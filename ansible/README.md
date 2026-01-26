How To:

Setup:

Setting uup Ansible was way easier than I was expecting. \
Pretty much you just do `apt install ansible` and it works out of the box. \
There are some other deployment options that I didn't explore because I just wnated to start using it, and they weren't really applicable to my setup or were more complicated than I felt they needed to be. \
Basically, you can also deploy Ansible via pip and run it in a venv, but I wanted a dedicated control node for my setup.\ 

So here's how I did the setup of my Ansible control node (based on my bash history): \

```
$ sudo apt install ansible
```

Confirm the install by checking the ansible version running on the machine:\ 
```
$ ansible --version
```

Create a directory for the ansible config, inventories, and playbooks \
I like to use `/etc` as the base for any config for any services on the machine because 1) that's what /etc/ is for, and 2) it helps ensure that there is some kind of standard to where my config files are kept for if I don't come back to the machine for some time.\ 
```
$ mkdir /etc/ansible
```

Inside of here I created the inventory file (I opted to use .yaml for this file, however you can also use INI. I just like YAML more), alongside a directory for playbooks and a basic playbook file to get me started:
```
$ touch /etc/ansible/inventory.yaml
$ mkdir /etc/ansible/playbooks
$ touch /etc/ansible/playbooks/k8s-node.yaml
```

A new user account needed to be configured on the machine to run all ansible functions against remote hosts as a dedicated user. \
I opted to keep it simple and created a user called `ansible` for this, which is also created on all nodes managed by ansible within my environment.
A SSH key was also created for this user account, and the public key is deployed to all managed nodes within my environment so that Ansible can connect securely without the need for a password to be parsed in the playbook command run on the control node (Even though Ansible salts ther password in your bash history so that it will never leak)
```
$ sudo useradd -m -g users ansible
# I also set the default shell for the user account for when I switch to this user account to run the ansible commands
$ usermod -s /bin/bash ansible
# Then I made the ansible user a sudo user so that it can run things with elevated permissions
$ sudo useradd ansible sudo
```

Next I needed to make a simple ansible inventory. \
As I was testing the ansible deployment against a single raspberry pi that I had set up for this purpose I started with that. \
My initial inventory file looked something like: \
```
k8s-worker:
  hosts:
    k8s-node-01.blackout.space:
      ansible_host: 10.1.42.54
  vars:
    ansible_connection: ssh
    ansible_ssh_user: ansible
    ansible_ssh_private_key_file: /home/ansible/.ssh/ansible-auth
```

This file creates a collection of nodes (`k8s-worker`), defines the address they can be reached on, and sets some connection specific config for these nodes. \
I define that they can be reached via SSH as user `ansible` using the specified private key on the control node. \
To add another host I simply added an item to the `hosts:` list: \
```
k8s-worker:
  hosts:
    k8s-node-01.blackout.space:
      ansible_host: 10.1.42.54
    k8s-node-02.blackout.space:
      ansible_host: 10.1.42.55
```

As I'll come to shortly in writing the playbook, defining host groups is important as it allows for writing playbooks that target specific host groups for specific tasks.

Once the inventory file was created I was able to test that the control node could run an arbitrary command against the node in the inventory file (thus confirming the config was correct) \
The below simply pings the machine, and returns `pong` if it is able to connect and execute the command: \
```
$ ansible -i /etc/ansible/inventory.yaml all -m ping
```

Then we head over to playbook creation. \
Intially I wanted to deploy some simple packages and update the apt cache. \
This was a simple enough test to ensure the control node could connect to the worker, and could run basic commands. It also let me see what the output looked like from a playbook run without performing any tasks which were not easily reverted. \
My initial playbook looked something similar to: \
```
- name: k8s-node playbook
  hosts: k8s-worker
  become: 'yes'
  tasks:
    - name: apt update
      apt:
        update_cache: yes

    - name: Install Required Packages
      apt:
        pkg:
          - vim
          - apt-transport-https
          - ca-certificates
          - curl
        state: latest
```

Finally to run and test the playbook I switch to the `ansible` user on the control node (mostly for accountability reasons) and run: \

```
$ ansible-playbook -i /etc/ansible/inventory.yaml /etc/ansible/playbooks/k8s-node.yaml
```



Resources used: \
https://spacelift.io/blog/ansible-kubernetes \
https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/ \
https://www.cherryservers.com/blog/install-kubernetes-ubuntu#prerequisites \
https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-ubuntu-20-04 \
https://medium.com/@RootRouteway/install-configure-ansible-26193ebc53fa \
https://docs.ansible.com/projects/ansible/devel/inventory_guide/intro_inventory.html
