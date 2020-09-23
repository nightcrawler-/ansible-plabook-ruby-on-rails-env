## How to deploy a Ruby on Rails application with Ansible, Capistrano and a CI/CD tool (Circle CI)


### Introduction

Ansible is an open-source software provisioning, configuration management, and application-deployment tool enabling infrastructure as code. It runs on many Unix-like systems, and can configure both Unix-like systems as well as Microsoft Windows.

Ansible is a tool used to manage server configurations to allow provisioning of servers without manually installing the required dependencies on the target virtual machines. 

#### Setup

Once you have created your Ubuntu virtual machine, you'll need to install python to enable Ansible to provision your server. The instruction below assume you have a user called `ubuntu` and you have downloaded the ssh keys, then `chmod 400 [your key]` to enable it's usage for ssh connection.

1. Install ansible `apt install ansible`
2. Create the project diretory and create an `inventory` file. The inventory file is essentially a list of files that need to be provisioned. In case multiple servers need to be configured, they can be configured in groups. The inventory file can be either in `YAML` file on an `INI` format. Create and name the file `production` or any relevant name depending on the environemt  you are trying to configure. Add the following lines to the file.
``` ; ansible/production
      ansible_host=hse.cafrecode.co.ke ansible_user=ubuntu
```
3. When using the `ansible` command, the `-i` (`--inventory`) flag will specify which inventory file to use, the `--private-key` to specify which private key file to use and `-m` to execute the ping module.
4. Run `ansible -i ansible/production --private-key your_key.pem -m ping rails`
