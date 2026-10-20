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

## Ansible variables
Ansible variables are placeholders that store values like strings, numbers, lists, or dictionary and make playbook dynamic instead of hardcoding values. 
### Variable Types
- String
- Number
- Boolean
- List
- Dictionary

### Registrying Variables
You can create variables from the output of an ansible task with the keyword _register_. You can use the registered variables in any later task in your play.

### Variable scoping
Variable scope defined the accessibility or visibility of a variable where it can be reached.
Ansible has three main scopes
- Global: This is set by config, environment variables and the command line
- Play: Each play and contained structures, vars entries (vars; vars_files; vars_prompt), role defaults and vars.
- Host: Variables directly associated to a host, like inventory, include_vars, facts or registered task outputs.

### Magic variables
The magic variable is a built-in, reserved system variable automatically created by Ansible to provide information about the current execution context, inventory structure, host status, and runtime environment.

#### Common Magic Variables
- hostvars: A dictionary containing variables and gathered facts assigned to any host in the play. You can use it to reference values or IP addresses from other servers.
- groups: A dictionary listing all inventory groups and the hostnames belonging to each group.
- group_names: A list of all the inventory groups that the current host is a member of.
- inventory_hostname: The name of the current host as defined in your inventory file (unlike ansible_hostname, which is a gathered fact from the remote system).
- inventory_hostname_short: The short version of inventory_hostname up to the first period (excluding the domain name).
- ansible_play_hosts: A list of all active hosts still participating in the current play.
- ansible_play_batch: A list of hostnames currently in scope for the active batch (controlled by the serial keyword).
- playbook_dir: The directory path where the currently executing playbook is located.
- inventory_dir & inventory_file: The directory path and filename of the inventory source being used.
- ansible_check_mode: A boolean (True or False) indicating whether Ansible is running with the --check flag.
- ansible_version: A dictionary detailing the running version of Ansible (full, major, minor, revision, string)

#### Ansible facts
Ansible facts are the information collected by Ansible regarding the managed hosts, including operating systems, IP addresses, attached filesystems, and more. 

```
playbook

  ansible.cfg

  [defaults]
host_key_checking = False

  inventory

localhost ansible_connection=local nameserver_ip=8.8.8.8 snmp_port=160-161
node01 ansible_host=node01 ansible_password=caleston123
node02 ansible_host=node02 ansible_password=caleston123
[web_nodes]
node01
node02

 

[all:vars]
app_list=['vim', 'sqlite', 'jq']
user_details={'username': 'admin', 'password': 'secret_pass', 'email': 'admin@example.com'}

 playbook.yaml

---
- name: 'Add nameserver in resolv.conf file on localhost'
  hosts: localhost
  become: true
  tasks:
    - name: 'Add nameserver in resolv.conf file'
      ansible.builtin.lineinfile:
        path: /tmp/resolv.conf
        line: 'nameserver '{{nameserver_ip}}'

===============================

---
- hosts: localhost

 vars:

   car_model: 'BMW M3'

   country_name: 'USA'

.  title: 'SYstems Engineer'
  tasks:
    - name: Print car model
      ansible.builtin.command: 'echo "My car is {{car_model}}'
    - name: Print country name
      ansible.builtin.command: 'echo "I live in the {{country_name}}"'
    - name: Print job title
      ansible.builtin.command: 'echo "I work as a {{title}}"'

========================
Inventory

 

localhost ansible_connection=local nameserver_ip=8.8.8.8 snmp_port=160-161
node01 ansible_host=node01 ansible_password=caleston123
node02 ansible_host=node02 ansible_password=caleston123
[web_nodes]
node01
node02

 

[all:vars]
app_list=['vim', 'sqlite', 'jq']
user_details={'username': 'admin', 'password': 'secret_pass', 'email': 'admin@example.com'}

 

app_install.yaml

---
- hosts: web_nodes
  become: true
  tasks:
    - name: Install applications
      ansible.builtin.dnf:
        name: "{{ item }}"
        state: present
      loop: '{{app_list}}'

 
Inventory

localhost ansible_connection=local nameserver_ip=8.8.8.8 snmp_port=160-161
node01 ansible_host=node01 ansible_password=caleston123
node02 ansible_host=node02 ansible_password=caleston123
[web_nodes]
node01
node02

 

[all:vars]
app_list=['vim', 'sqlite', 'jq']
user_details={'username': 'admin', 'password': 'secret_pass', 'email': 'admin@example.com'}

 

User_setup.yaml

---
- hosts: all
  become: true
  tasks:
    - name: Set up user
      ansible.builtin.user:
        name: '{{user_details.username}}'
        password: '{{user_details.password}}'
        comment: '{{user_details.email}}'
        state: present
```

## Ansible Playbook
A playbook is a single YAML file that holds a set of plays.

- verify the playbook to catch error or unexpected behavior before run it. This can be done by
  - check mode - dry run (ansible-playbook playbook.yaml --check)
  - diff mode - (ansible-playbook playbook.yaml --check --diff)
  - Syntax check - (ansible-playbook playbook.yaml --syntax-check)

### Ansible Lint
A code reviwer for the playbook using _ansible-lint playbook.yaml_

## Ansible Module
Ansible modules are reusable, standalone scripts that Ansible copies to and executes on managed nodes or via APIs to perform specific tasks

### Common Types & Examples
- _System Modules_: Manage operating system elements like users, groups, and services (e.g., user, group, service, systemd).
- _Packaging Modules_: Handle software installation across different Linux distributions (e.g., ansible.builtin.apt, ansible.builtin.dnf, pip).
- _File Modules_: Handle copying files, setting permissions, and managing directories (e.g., copy, file, template).
- _Cloud & API Modules_: Provision and manage resources on platforms like AWS, Azure, or Google Cloud (e.g., amazon.aws.ec2_instance).

### How They Work
1. Invocation: You call a module inside an Ansible task (within a playbook) or via an ad-hoc command.
2. Transfer: Ansible packages the module and sends it to a temporary location on the remote managed node (over SSH or WinRM).
3. Execution: The module runs locally on the managed node, executes the action, and prints a JSON result.
4. Cleanup: Ansible removes the temporary script file from the remote node.

### Ansible plugin
Ansible plugins are pieces of reusable code that augument or extend Ansible core functionality.

#### Common Types of Ansible Plugins
- _Connection Plugins_: Control how Ansible communicates with target hosts (e.g., using native ssh, paramiko, local, or docker). 
- _Lookup Plugins_: Fetch and pull data from external sources (like reading a local file, checking environment variables, or querying databases) into your playbooks. 
- _Filter Plugins_: Manipulate and format data inside your playbooks and Jinja2 templates (e.g., parsing strings or transforming JSON data). 
- _Inventory Plugins_: Dynamically construct the host inventory from various cloud infrastructure providers or data sources (such as an AWS EC2 instance list) instead of relying on a static text file. 
- _Callback Plugins_: Hook into Ansible's execution lifecycle to capture events. They are commonly used to format terminal output (like printing in JSON) or send notifications to third-party tools like Slack. 
- _Vars Plugins_: Inject extra variable data into Ansible runs that do not originate from an inventory or a command-line argument (this is how internal features like host_vars and group_vars operate). 
- _Action Plugins_: Act as front-ends to modules, allowing processing to happen on the controller machine before the actual module executes on the target node. 
- _Module plugins_

## Ansible Handler
In Ansible, handlers are special tasks that only run when triggered (notified) by another task, and only if that task actually makes a change to the system for example: the Nginx service will only reload if the ansible.builtin.template task actually modifies the nginx.conf file on the remote server
```
---
- name: Configure Web Server
  hosts: webservers
  become: yes

  tasks:
    - name: Copy Nginx configuration file
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx  # This must exactly match the handler name

  handlers:
    - name: Restart Nginx
      ansible.builtin.service:
        name: nginx
        state: restarted

```
## Ansible Roles
Ansible roles let you package and organize tasks, variables, files, templates, and handlers into a standardized directory structure for easy reusability.

### Standard Directory Structure
A role organizes automation code into specific subfolders:
- _tasks/main.yml_: The main list of tasks executed by the role.
- _handlers/main.yml_: Event-driven tasks (like restarting a service) triggered by notifications.
- _defaults/main.yml_: Low-priority default variables that can be overridden easily.
- _vars/main.yml_: High-priority variables that are difficult to override.
- _files/_: Static files that need copying to the target nodes.
- _templates/_: Dynamic Jinja2 configuration templates.
- _meta/main.yml_: Metadata, author info, and role dependencies.

### How to Create and Use a Role
1. Initialize the structure: Run the Galaxy command to generate the standard folders:
```
bash
ansible-galaxy role init my_role
```
Use code with caution.
2. Add tasks: Write your automation logic inside my_role/tasks/main.yml.
3. Call the role in a playbook: Reference it under the roles keyword:yaml
```
- hosts: webservers
  roles:
    - my_role
```
Use code with caution.

### Best Practices

- Single Responsibility: Keep roles focused on doing one thing well (e.g., separate nginx and mysql roles instead of an all-in-one role). Users on the Ansible Forum generally reach a consensus that small, generic, single-task roles improve maintainability.
- Use Defaults: Define customizable variables in defaults/ rather than vars/ so other users can override them without fighting variable precedence.
### Ansible collection

Ansible collections are a standard packaging and distribution format for Ansible content—including playbooks, roles, modules, and plugins—that allow you to organize, version, and share automation code independently of core Ansible releases.