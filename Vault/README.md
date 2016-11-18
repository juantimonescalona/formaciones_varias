#What is Vault?

Vault is a tool for securely accessing secrets. 
A secret is anything that you want to tightly control access to, such as API keys, passwords, certificates, and more. 

Vault provides a unified interface to any secret, while providing tight access control and recording a detailed audit log.
A modern system requires access to a multitude of secrets: database credentials, API keys for external services, credentials for service-oriented architecture communication, etc. Understanding who is accessing what secrets is already very difficult and platform-specific. Adding on key rolling, secure storage, and detailed audit logs is almost impossible without a custom solution. This is where Vault steps in.
Examples work best to showcase Vault. Please see the use cases.

#The key features of Vault are:

##Secure Secret Storage: 
Arbitrary key/value secrets can be stored in Vault.
Vault encrypts these secrets prior to writing them to persistent storage, 
so gaining access to the raw storage isn't enough to access your secrets. Vault can write to disk, Consul, and more.

##Dynamic Secrets: 
Vault can generate secrets on-demand for some systems, such as AWS or SQL databases. 
For example, when an application needs to access an S3 bucket, it asks Vault for credentials, 
and Vault will generate an AWS keypair with valid permissions on demand. 
After creating these dynamic secrets, Vault will also automatically revoke them after the lease is up.

#Data Encryption: 
Vault can encrypt and decrypt data without storing it. 
This allows security teams to define encryption parameters and 
developers to store encrypted data in a location such as SQL without having to design their own encryption methods.

##Leasing and Renewal: 
All secrets in Vault have a lease associated with them. 
At the end of the lease, Vault will automatically revoke that secret. Clients are able to renew leases via built-in renew APIs.

##Revocation: 
Vault has built-in support for secret revocation. 
Vault can revoke not only single secrets, but a tree of secrets, 
for example all secrets read by a specific user, or all secrets of a particular type. 
Revocation assists in key rolling as well as locking down systems in the case of an intrusion.

# Ansible Vault module
New in Ansible 1.5, “Vault” is a feature of ansible that allows keeping sensitive data such as passwords or keys in encrypted files, 
rather than as plaintext in your playbooks or roles. These vault files can then be distributed or placed in source control.

To enable this feature, a command line tool, ansible-vault is used to edit files, and a command line flag –ask-vault-pass or 
–vault-password-file is used. Alternately, you may specify the location of a password file or command Ansible to always prompt for the 
password in your ansible.cfg file. These options require no command line flag usage.

For best practices advice, refer to Variables and Vaults.

##Creating Encrypted Files
To create a new encrypted data file, run the following command:
```
$ ansible-vault create foo.yml
```
First you will be prompted for a password. 
The password used with vault currently must be the same for all files you wish to use together at the same time.

##Editing Encrypted Files
```
$ ansible-vault edit foo.yml
```

##Rekeying Encrypted Files
```
$ ansible-vault rekey foo.yml bar.yml
```
This command can rekey multiple data files at once and will ask for the original password and also the new password.

##Decrypting Encrypted Files
```
$ ansible-vault decrypt foo.yml bar.yml
```

##Viewing Encrypted Files
```
$ ansible-vault view foo.yml bar.yml
```

##Running a Playbook With Vault
```
$ ansible-playbook site.yml --ask-vault-pass
```

Alternatively, passwords can be specified with a file or a script, the script version will require Ansible 1.7 or later. When using this flag, 
ensure permissions on the file are such that no one else can access your key and do not add your key to source control:
```
ansible-playbook site.yml --vault-password-file ~/.vault_pass.txt

ansible-playbook site.yml --vault-password-file ~/.vault_pass.py
```
The password should be a string stored as a single line in the file.

You can also set ```ANSIBLE_VAULT_PASSWORD_FILE``` environment variable, e.g. ```ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass.txt``` and Ansible 
will automatically search for the password in that file.

##Speeding Up Vault Operations
By default, Ansible uses PyCrypto to encrypt and decrypt vault files. If you have many encrypted files, 
decrypting them at startup may cause a perceptible delay. To speed this up, install the cryptography package:
```
  pip install cryptography
```
