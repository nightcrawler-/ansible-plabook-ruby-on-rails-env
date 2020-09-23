## How to deploy a Ruby on Rails application with Ansible, Capistrano and a CI/CD tool (Circle CI)


### Introduction

This document summaries the content of this repository. Some information is ommited but may be found on the appropriate configuration files

Ansible is an open-source software provisioning, configuration management, and application-deployment tool enabling infrastructure as code. It runs on many Unix-like systems, and can configure both Unix-like systems as well as Microsoft Windows.

Ansible is a tool used to manage server configurations to allow provisioning of servers without manually installing the required dependencies on the target virtual machines. 

#### Setup

Once you have created your Ubuntu virtual machine, you'll need to install python (`sudo apt-get install python`) to enable Ansible to provision your server. The instruction below assume you have a user called `ubuntu` and you have downloaded the ssh keys, then `chmod 400 [your key]` to enable it's usage for ssh connection.

Install ansible `apt install ansible`

Create the project diretory and create an `inventory` file. The Inventory is a description of the nodes that can be accessed by Ansible. In case multiple servers need to be configured, they can be configured in groups. The inventory file can be either in `YAML` file on an `INI` format. Create and name the file `production` or any relevant name depending on the environemt  you are trying to configure. Add the following lines to the file.

``` ; ansible/production
      ansible_host=hse.cafrecode.co.ke ansible_user=ubuntu
```
When using the `ansible` command, the `-i` (`--inventory`) flag will specify which inventory file to use, the `--private-key` to specify which private key file to use and `-m` to execute the ping module.
Run `ansible -i ansible/production --private-key your_key.pem -m ping rails`.You should get the below response if everything is successful:

```
rails | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

##### Setting up the server

###### Playbooks

Playbooks are YAML files that express configurations, deployment, and orchestration in Ansible, and allow Ansible to perform operations on managed nodes. Each Playbook maps a group of hosts to a set of roles. Each role is represented by calls to Ansible tasks.

Create a playbook file, `app.yml` that will state the configuration needs to be executed on the `rails` server defined in the inventory file, and all actions should be run as the `root` user.


```
# ansible/app.yml

---
- hosts: rails
  remote_user: root
  become: yes
  tasks:
```

 The first task is going to be upgrading all packages on the server to the latest versions

```
# ansible/app.yml

---
- hosts: rails
  remote_user: root
  become: yes
  tasks:
  	- name: Upgrade packages
  	  apt: 
  	  	uprade: dist

```

 Then run: ansible-playbook -i ansible/production --private-key ~/Dev/cafrecode/ssh/fno-dev.pem ansible/app.yml

 Once finished the packages should be updated.

###### Add deploy user

Add this task after the above one

```
    - name: Add deploy user
        user:
          name: deploy
          shell: /bin/bash
```

Create an ssh key to authenticate the deploy user. If one is already present on your machine, this step can be skipped.

` $ ssh-keygen -t rsa -b 4096 -C "your_email@example.com" `

Next, add the key to the ssh-agent
 ```
$ eval "$(ssh-agent -s)"
$ ssh-add -K ~/.ssh/id_rsa

```

Add a task to add the public key to the deploy user on the server:

```
- name: Add SSH key to server for deploy user
      authorized_key:
        user: deploy
        key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuxMbjZLEk0AqHkl8gukJHWNEIB8UP3HRzmE6UoYP6KxtbDaekDTZ10COHzzO0Vo3C9If9v9UmOkggBOWmF8GTurcFR/p70POeuyw9dmpcCm72dTCadOJCHB2m/vPTyhA14P7nXruLk9nQSIsIoaRQlG5/p/6kMY3jUHIhOVLiJUOzE2vpl8XGYZzzewqZpF/9jRl1nTRoL0XFtoCoUVjAY6jMt5wHp7Gl8/1tEFkiab9JvX0oj5zAOSxWpqLI+32U2wnwvbEniHNt2ZC+PPykA3P744RMQmrSADpVk2f2J5kwUt2W1xZOpUbGE9FMyOaD5i1Tq1rSmBTZa9RR1Ekd frederick@frederick-hp"
 ```

 Install Ruby on Rails dependencies

 ```
 # ansible/site.yml

...
  tasks:
    ...
    - name: Install Ruby dependencies
      apt:
        name: "{{ item }}"
      with_items:
        - gcc
        - autoconf
        - bison
        - build-essential
        - libssl-dev
        - libyaml-dev
        - libreadline6-dev
        - zlib1g-dev
        - libncurses5-dev
        - libffi-dev
        - libgdbm3
        - libgdbm-dev
        - sqlite3
        - libsqlite3-dev
        - nodejs
 ```


