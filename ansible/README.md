How To:

To confirm the Control Node can talk to other nodes: \
Exec `ansible -i {/path/to/inventory/file} all -m ping`

To run the Ansible playbook: \
On the Control Node login as the ansible user (This is done mostly for accountability tracking) \
Exec `ansible-playbook -i {/path/to/inventory/file} {/path/to/playbook}`
The ansible playbook will then run across all nodes within the inventory file



Resources used: \
https://spacelift.io/blog/ansible-kubernetes \
https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/ \
https://www.cherryservers.com/blog/install-kubernetes-ubuntu#prerequisites \
https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-ubuntu-20-04 \
https://medium.com/@RootRouteway/install-configure-ansible-26193ebc53fa \
https://docs.ansible.com/projects/ansible/devel/inventory_guide/intro_inventory.html
