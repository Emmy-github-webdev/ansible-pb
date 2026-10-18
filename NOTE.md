# Ansible
Ansible is an open-source IT automation platform used for configuration management, software deployment, cloud provisioning and workflow orchestration.

## Default Locations
- _/etc/ansible/ansible.cfg_
- Invetory - _/etc/ansible/host_(Host list)
- Log_path - _/var/log/ansible.log_(Where logs goes)
- roles_path - _/etc/ansible/roles_(Where roles live)
- gathering - _implicit_(gather facts)
- timeout - _10_(SSH time out)
- forks - _5 -> 20_(Number of host it can handle at once)

## Order of Precedence
Ansible offers fours sources for controlling it's behavior
In order of precedence from lowest (most easily overridden) to highest (overrides all others), the categories are:
- _Configuration settings_
- _Command-line options_
- _Playbook keywords_
- _Variables_

Direct Assignment
1. Configuration settings: Include both value from ansible.cfg file and environmet variables. Within this category, values set in configuration files have lower precedence. Ansible used the first config file it finds and ignoring all others.
- Environment variable _ANSIBLE_CONFIG_ - _highest_
- Current directory _./ansible.cfg_     - _Then_
- Home directory _~/.ansible.cfg_ - _Then_
- Default location _/etc/ansible/ansible.cfg_ - _Last_

2. Command-line Options: Any command-line option will override any configuration setting only. They do not override playbook keyword, variables from inventory or variables from playbooks. It has low precedence.

3. Playbook keyword: Any playbook keyword (Play, Role, Block, Task) will override any command-line option and any configuration setting.

4. Variables: Ansible variables are high on the precedence stack. They override any playbook keyword, command-line option, environment variable and any configuration file setting.

## Inspect with ansible-config
- List ansible config - _ansible-config-list_
- DEFAULT_FORKS - 5, TYPE: integer
- View ansible config - _ansible-config view_
- Dump - _ansible-config dump_

## Ansible Inventory
An Ansible inventory is a file, script, or directory that defines a collection of managed nodes and network locations so that Ansible can connect to them, group them, and run automation tasks against them. Inventory uses ini format

### How Ansible Connects
Ansible connects using _ansible_connect_
- For Linux machine - _ansible_connect=ssh_
- For windows machine - _ansible_connect=winrm_

Different ways of adding managed node in inventory are
1. Individually
```
web server1.company.com
db server2.company.com
``` 
2. As agroup
```
[mail]
server3.company.com
server4.company.com

[db]
server5.company.com
server6.company.com
```
3. Other way
```
myhosts:
  hosts:
    my_host_01:
      ansible_host: 192.0.2.50
    my_host_02:
      ansible_host: 192.0.2.51
    my_host_03:
      ansible_host: 192.0.2.52
```
### Example
```
playbook
  - ansible.cfg
    [default]
    host_key_checking = false

  - inventory
  # Sample inventory file
  web1 ansible_host=server1.company.com
  web2 ansible_host=server2.company.com
  web3 ansible_host=server3.company.com
  db1 ansible_host=server4.company.com

  or wit user and password

  # Web Servers
  web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_password=Password123!
  web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root ansible_password=Password123!
  web3 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_password=Password123!

  # Database Servers
  db1 ansible_host=server4.company.com ansible_connection=winrm ansible_user=administrator ansible_password=Dbp@ss123!

  Or by grouping

  # Web Servers
  web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_password=Password123!
  web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root ansible_password=Password123!
  web3 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_password=Password123!

  # Database Servers
  db1 ansible_host=server4.company.com ansible_connection=winrm ansible_user=administrator ansible_password=Password123!


  [web_servers]
  web1
  web2
  web3

  [db_servers]
  db1

  # parent_group:children is creating a nested group (or parent group) that contains other child groups 
  [parent_group:children]
  web_servers
  db_servers

  Or for more administration
  # Web Servers
web_node1 ansible_host=web01.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
web_node2 ansible_host=web02.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
web_node3 ansible_host=web03.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass

# DB Servers
sql_db1 ansible_host=sql01.xyz.com ansible_connection=ssh ansible_user=root ansible_password=Lin$Pass
sql_db2 ansible_host=sql02.xyz.com ansible_connection=ssh ansible_user=root ansible_password=Lin$Pass

[db_nodes]
sql_db1
sql_db2

[web_nodes]
web_node1
web_node2
web_node3

[boston_nodes]
sql_db1
web_node1

[dallas_nodes]
sql_db2
web_node2
web_node3

[us_nodes:children]
boston_nodes
dallas_nodes
```

### Inventory format
Inventory formats are _ini_ and _yaml_ format
- INI
```
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com
db2.example.com
```
- YAML
```
all:
  children:
    webservers
      hosts:
       web1.example.com
       web2.example.com   

    dbservers"
      hosts:
       db1.example.com
       db2.example.com 
```
1. A startup with a few server
- Web Hosting
- Database

2. Global company with hundresa of servers