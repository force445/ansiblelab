# Ansible Installation

```bash
#Install python3 and pip on Ubuntu:

sudo apt update

sudo apt install python3-pip

#Validate you have python and pip installed:

Python3 ---version

pip ---version

#Install Ansible using pip:

pip install ansible

#Validate Ansible

Ansible ---version

```
# Ansible tutorial
This tutorial will deploy the nginx to the managed node.
First we declare the managed node in `inventory.ini` or `inventory.yaml`
```ini
#inventory.ini
[hosts]
node01 ansible_host=192.168.122.182 ansible_connection=ssh ansible_user=force ansible_ssh_private_key_file=~/ansiblelab/test ansible_become=true ansible_become_method=sudo ansible_become_user=root ansible_become_password=Force4462

node02 ...
node03 ...
```
node01: The alias for the managed node.

ansible_host: The IP address or hostname of the managed node.

ansible_connection: The connection type to use. In this case, ssh is specified.

ansible_user: The SSH user to connect as.

ansible_ssh_private_key_file: The path to the SSH private key file used for authentication.

ansible_become: Indicates whether privilege escalation (become) is used. true means it is enabled.

ansible_become_method: The method used for privilege escalation. sudo is specified here.

ansible_become_user: The user to become after privilege escalation. root is specified here.

ansible_become_password: The password for the become user.

To verify hosts use the command.
```bash
ansible -i inventory.ini all -m ping
```
If the hosts are correctly configured and reachable, you should see output similar to this:

```bash
node01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
node02 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
node03 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
If there are issues, the output will provide error messages indicating what went wrong, such as SSH connection failures or authentication issues.

## Create playbook

After declare inventory we create the playbook by create file `playbook.yaml` .
Ansible Playbooks are the heart of Ansible's configuration management and automation capabilities. They are written in YAML and describe a series of steps (or "plays") that Ansible should execute on the managed nodes. Each play consists of tasks that define the desired state of the system.

```yml
---
  - name: Install nginx 
    hosts: all
    become: true
    tasks:
      - name: Install nginx
        apt:
          name: nginx
          state: latest
      - name: Start nginx
        service:
          name: nginx
          state: started
      - name: Allow Uncomplicated Firewall
        ufw:
          rule: allow
          port: '80'
          proto: tcp
          state: enabled
      - name: Copy newindex.html to public index.html
        ansible.builtin.copy:
          src: ~/ansiblelab/newindex.html
          dest: /var/www/html/index.nginx-debian.html
          follow: yes
      - name: Restart nginx
        ansible.builtin.service:
          name: nginx
          state: reloaded
```

Here is a breakdown of each section:
```yml
- name: Install nginx 
  hosts: all
  become: true
```
name: Describes the play.

hosts: Specifies the target hosts. In this case, all means all hosts in the inventory.

become: Enables privilege escalation (sudo).

```yml
- name: Install nginx
  apt:
    name: nginx
    state: latest
```
name: Describes the task.

apt: Uses the apt module to manage packages.

name: Specifies the package to install (nginx).

state: Ensures the package is installed and up-to-date (latest).

```yml
- name: Start nginx
  service:
    name: nginx
    state: started
```
name: Describes the task.

service: Uses the service module to manage services.

name: Specifies the service to manage (nginx).

state: Ensures the service is running (started).

```yml
- name: Allow Uncomplicated Firewall
  ufw:
    rule: allow
    port: '80'
    proto: tcp
    state: enabled
```
name: Describes the task.

ufw: Uses the ufw module to manage firewall rules.

rule: Specifies the rule action (allow).

port: Specifies the port to allow (80).

proto: Specifies the protocol (tcp).

state: Ensures the rule is enabled (enabled).

```yml
- name: Copy newindex.html to public index.html
  ansible.builtin.copy:
    src: ~/ansiblelab/newindex.html
    dest: /var/www/html/index.nginx-debian.html
    follow: yes
```
name: Describes the task.

ansible.builtin.copy: Uses the copy module to copy files.

src: Specifies the source file path.

dest: Specifies the destination file path.

follow: Ensures symbolic links are followed.

```yml
- name: Restart nginx
  ansible.builtin.service:
    name: nginx
    state: reloaded
```
name: Describes the task.

ansible.builtin.service: Uses the service module to manage services.

name: Specifies the service to manage (nginx).

state: Ensures the service is reloaded (reloaded).

## Run the playbook
after we have inventory and playbook then we will run the play book by using the command:

```bash
ansible-playbook -i inventory.ini playbook.yaml
```
If everything is correct the output should be like this

```bash
PLAY [Install nginx] ***********************************************************

TASK [Gathering Facts] *********************************************************
[WARNING]: Platform linux on host node01 is using the discovered Python
interpreter at /usr/bin/python3.10, but future installation of another Python
interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [node01]

TASK [Install nginx] ***********************************************************

changed: [node01]

TASK [Start nginx] *************************************************************
ok: [node01]

TASK [Allow Uncomplicated Firewall] ********************************************
ok: [node01]

TASK [Copy newindex.html to public index.html] *********************************
changed: [node01]

TASK [Restart nginx] ***********************************************************
changed: [node01]

PLAY RECAP *********************************************************************
node01                     : ok=6    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
