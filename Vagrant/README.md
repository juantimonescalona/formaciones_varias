#Uso básico de Vagrant
##¿Qué es Vagrant?
Vagrant es una herramienta de HashiCorp que sirve para crear entornos de trabajo portables y ligeros
para el desarrollo, usando automatizaciones con scripts se puede levantar máquinas virtuales con distintos
sistemas operativos y configuraciones. 
Vagrant permite acoplarse a nuestros playbooks de Ansible para realizar test automáticos, simulando estar en
una máquina con el SO igual que en el que queremos instalar nuestro ansible-playbook.

##Como usamos Vagrant manualmente
Vagrant tiene un repositorio con los sistemas operativos que podemos levantar y que han sido ya creados por otros
desarrolladores. Siempre es importante consultar si lo que necesitamos ya está creado para perder el menor tiempo posible.

###HashiCorp Atlas 
URL: https://atlas.hashicorp.com/boxes/search
Podemos elegir el proveedor donde queremos instalar la máquina virtual, yo para mis pruebas locales uso Virtualbox
Un ejemplo seria un ubuntu 14.04 x64 : https://atlas.hashicorp.com/puphpet/boxes/ubuntu1404-x64

``` html
Switching to Bentos build configs.

Generated from https://github.com/puphpet/bento

vmware_desktop
    vagrant init puphpet/ubuntu1404-x64; vagrant up --provider vmware_desktop
    
virtualbox
    vagrant init puphpet/ubuntu1404-x64; vagrant up --provider virtualbox
    
parallels
    vagrant init puphpet/ubuntu1404-x64; vagrant up --provider parallels
    
3 providers for this version.

```

*** Todos los comandos de vagrant hay que ejecutarlos en el path donde hemos descargado el Vagrantfile con el que levantamos la vm

Ya nos vendrían los comandos hasta para levantar la máquina virtual en 3 proveedores, 
esto bajaria al path donde estamos todos los archivos necesarios para generar automaticamente la maquina virtual, 
que basicamente es el Vagrantfile y una carpeta .vagrant con los metadatos.
Podemos modificar manualmente el Vagrantfile y añadir o modificar parametros de la máquina virtual como la memoria ram
y actualizar luego la máquina con los cambios usando: ```vagrant reload```

Por ejemplo he agregado:
- Un nombre concreto a mi maquina virtual para localizarla en la gui de Virtualbox: ```ubuntu1404-x64```
- Asigne 2048mb de ram a la máquina virtual
- Asigne una red privada y una ip fija que no esta cogida para entrar a ella por ssh (Esto último en principio no haria falta puesto que Vagrant tiene el comando ```vagrant ssh```)
```Vagrantfile
  config.vm.provider "virtualbox" do |vb|
     vb.name = "ubuntu1404-x64"
     vb.memory = "2048"
   end

  config.vm.hostname = "ubuntu1404-x64"
  config.vm.network :private_network, ip: "192.168.21.112"
```

###Comandos esenciales Vagrant
Como hemos dicho anteriormente, hay que ejecutarlos sobre la ruta donde tenemos nuestro Vagrantfile

```
     $ vagrant [options] <command> [<args>]
      
     COMMAND              HELP
     ------------------   -------------------------------------------------------------------
     destroy              stops and deletes all traces of the vagrant machine
     global-status        outputs status Vagrant environments for this user
     halt                 stops the vagrant machine
     init                 initializes a new Vagrant environment by creating a Vagrantfile
     port                 displays information about guest port mappings
     reload               restarts vagrant machine, loads new Vagrantfile configuration
     resume               resume a suspended vagrant machine
     snapshot             manages snapshots: saving, restoring, etc.
     ssh                  connects to machine via SSH
     ssh-config           outputs OpenSSH valid configuration to connect to the machine
     status               outputs status of the vagrant machine
     suspend              suspends the machine
     up                   starts and provisions the vagrant environment
     ------------------   -------------------------------------------------------------------
```

##Como usamos Vagrant en nuestro ansible-playbook
Vamos a usar un ejemplo bajado de Github para explicarlo rápidamente.
Encontré este repositorio con un monton de ejemplos: https://github.com/geerlingguy/ansible-vagrant-examples.git

Por ejemplo vamos a testear el ejemplo con una instalacion de jenkins usando Vagrant, que he copiado del proyecto original a este.

El archivo Vagrantfile es el siguiente
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "geerlingguy/ubuntu1604"
  config.ssh.insert_key = false
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider :virtualbox do |v|
    v.name = "jenkins"
    v.memory = 512
    v.cpus = 2
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
  end

  config.vm.hostname = "jenkins"
  config.vm.network :private_network, ip: "192.168.33.55"

  # Set the name of the VM. See: http://stackoverflow.com/a/17864388/100134
  config.vm.define :jenkins do |jenkins|
  end

  # Ansible provisioner.
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provisioning/playbook.yml"
    ansible.inventory_path = "provisioning/inventory"
    ansible.sudo = true
  end

end
```

Cambios: 
    la ip --> config.vm.network :private_network, ip: "192.168.21.111"
    la ram -->  v.memory = 1024
    
Los ansible-playbooks se meten dentro del Vagrantfile

```
  # Ansible provisioner.
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provisioning/playbook.yml"
    ansible.inventory_path = "provisioning/inventory"
    ansible.sudo = true
  end
```

Basicamente esta ejecutando ansible especificando el playbook(tareas a ejecutar con ansible) y el inventory (destinos) con permisos de sudo

En el inventory hemos modificado la ip igual que la que le pusimos en el vagrantfile -->  ansible_ssh_host=192.168.21.111

En el playbook se hace referencia a otros roles que son necesarios
```
  roles:
    - geerlingguy.firewall
    - geerlingguy.ntp
    - geerlingguy.git
    - geerlingguy.java
    - geerlingguy.jenkins
```
Y consulta cuales son sus sources en el archivo requeriments.yml
```
---
- src: geerlingguy.firewall
- src: geerlingguy.ntp
- src: geerlingguy.git
- src: geerlingguy.java
- src: geerlingguy.jenkins  #https://galaxy.ansible.com/geerlingguy/jenkins/ --> https://github.com/geerlingguy/ansible-role-jenkins
```
Esta comentado como busca en el galaxy un rol, usando por ejemplo el ultimo:

src: geerlingguy.jenkins  --> se transforma en  -->https://galaxy.ansible.com/geerlingguy/jenkins/ --> cuyo código está en --> https://github.com/geerlingguy/ansible-role-jenkins

Al ejecutar el Vagrantfile nos bajaria todos los roles necesarios y los ejecutaría en nuestra máquina virtual.

Para ejecutar este ejemplo tenemos los dos pasos README.md :

Run the following command to install the necessary Ansible roles for this profile: ```$ ansible-galaxy install -r requirements.yml```

Once all of that is done, you can simply type in ```vagrant up``` , 
and Vagrant will create a new VM, install the base box, and configure it.
