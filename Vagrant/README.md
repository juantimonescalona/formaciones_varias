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

ya nos vendrían los comandos hasta para levantar la máquina virtual en 3 proveedores, 
esto bajaria al path donde estamos todos los archivos necesarios para generar automaticamente la maquina virtual, 
que basicamente es el Vagrantfile y una carpeta .vagrant con los metadatos.
Podemos modificar manualmente el Vagrantfile y añadir o modificar parametros de la máquina virtual como la memoria ram
y actualizar luego la máquina con los cambios usando: ```vagrant reload```