In this lab exercise, you will use the hosts below:

student-node :- This host will act as an Ansible master node where you will create playbooks, inventory, roles, etc., and you will be running your playbooks from this host itself.

node01 :- This host will act as an Ansible client/remote host where you will set up/install some stuff using Ansible playbooks. Below are the SSH credentials for this host:

User: bob
Password: caleston123
node02 :- This host will also act as an Ansible client/remote host where you will set up/install some stuff using Ansible playbooks. Below are the SSH credentials for this host:

User: bob
Password: caleston123
Note: Please type exit or logout on the terminal or press CTRL + d to log out from a specific node.

playbook
---
- hosts: localhost
  tasks:
    - name: Print car model
      ansible.builtin.command: 'echo "My car is BMW M3"'
    - name: Print country name 
      ansible.builtin.command: 'echo "I live in the USA"'
    - name: Print job title
      ansible.builtin.command: 'echo "I work as a Systems Engineer"':wq!
      