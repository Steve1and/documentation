# Ansible Notes
***

https://docs.ansible.com/ansible/latest/user_guide/
https://www.tutorialspoint.com/ansible/

- Ansible is an automated deployment engine that doesn't use any agents or custom security infrastructure.
- Ansible uses playbooks to describe automation jobs and are written in YAML format.
- Ansible connects all of a network's nodes via ssh (by default). It can also use Kerberos for Windows targets.
- After connecting to a network's nodes, Ansible pushes small programs called "Ansible Modules". These modules are removed once finished running.
- Ansible manages a network inventory in simple text files (the host file).
- **Sample Host File**:
```
#File name: hosts
#Description: Inventory file for your application. Defines machine type abc node to deploy specific artifacts
# Defines machine type def node to upload metadata.

[abc-node]
#server1 ansible_host = <target machine for DU deployment> ansible_user = <Ansible user> ansible_connection = ssh server1 ansible_host = <your host name> ansible_user = <your unix user> ansible_connection = ssh

[def-node]
#server2 ansible_host = <target machine for artifact upload> ansible_user = <Ansible user> ansible_connection = ssh server2 ansible_host = <host> ansible_user = <user> ansible_connection = ssh
```
- The Ansible Host file is typically stored at: `/etc/ansible/hosts`
***

## Installation Process

There are two types of machines when referring to Ansible deployment:
- **Control Machine**: Machine from where you can manage other machine.
- **Remote Machine**: Machines which are handled/controlled by control machine.

Ansible can be run from any machine with Python 2 (versions 2.6 or 2.7) or Python 3 (versions 3.5 and higher) installed.
- **NOTE**: Windows does not support being a control machine.

Ansible does not add any database. It does not require any daemons to start or keep it running.

To install Ansible on an Ubuntu Machine, do the following:
- `sudo apt-get update`
- `sudo apt-get install software-properties-common`
- `sudo -E apt-add-repository ppa:ansible/ansible`
- `sudo apt-get update`
- `sudo apt-get install ansible`
***

## YAML Basics

Every YAML file optionally starts with `---` and ends with `...`.

**Key-Value Pair**: YAML uses simple key-value pair to represent the data. The dictionary is represented in key: value pair.
- **NOTE**: There should be a space between : and value.
- **Example**:
  ```
  --- #Optional YAML start syntax
  user:
    name: user
    UUID: 500
    GUID: 500
  ... #Optional YAML end syntax
  ```
- You can also use abbreviation to represent dictionaries:
  - `user: {name: user, UUID: 500, GUID: 500}`

**Representing List**: Lists can be represented in YAML where every element (member) of the list should be written in a new line with the same indentation starting with `- `.
- **Example**:
  ```
  ---
  users:
    - user
    - user2
    - user3
  ...
  ```
- You can also use abbreviation to represent lists:
  - `users: ['user', 'user2', 'user3']`
- Lists can also be used inside dictionaries:
  ```
  ---
  user:
    name: user
    UUID: 500
    GUID: 500
    groups:
      - standard_users
      - tier_one_admins
  ...
  ```

**List of Dictionaries**: YAML files can have lists of dictionaries.
- **Example**:
  ```
  ---
  user:
    name: user
    UUID: 500
    GUID: 500
    groups:
      - standard_users
      - tier_one_admins

  user2:
    name: user2
    UUID: 501
    GUID: 501
    groups:
      - standard_users
      - tier_two_admins
  ...
  ```
- YAML uses `|` to include newlines while showing multiple lines and `>` to suppress newlines while showing multiple lines.
- YAML represents Boolean (true/false) values:
  ```
  authorized: TRUE

  messageIncludeNewLines: |
    You're In!

  messageExcludeNewLines: >
    You're In!
  ```
***

## Ad hoc Commands

**Parallelism and Shell Commands**: In this example a target machine is rebooted in 12 parallel forks at a time.
1. Set up `ssh-agent` for the connection:
  - `ssh-agent bash`
  - `ssh-add ~/.ssh/id_rsa`
2. To run reboot for all the servers in group `one` in 12 parallel forks, run the following command:
  - `Ansible one -a "/sbin/reboot" -f 12`
3. By default, Ansible will run the above Ad-hoc commands from the current user account. To change this behavior, a username can be passed with the `-u` flag:
  - `Ansible one -a "/sbin/reboot" -f 12 -u user`

**File Transfer**: Use ad-hoc commands to `scp` files in parallel on multiple machines:
- Transfer files to many servers/machines:
  - `Ansible one -m copy -a "src=/etc/yum.conf dest=/tmp/yum.conf"`
- Creating a new directory:
  - `Ansible one -m file -a "dest=/home/user/new mode=777 owner=user group=user state=directory"`
- Deleting whole directories and files:
  - `Ansible one -m file -a "dest=/home/user/new state=absent"`

**Managing Packages**:
- Check if a yum package is installed or not, but do not update:
  - `Ansible one -m yum -a "name=demo-tomcat-1 state=present"`
- Check if the package is not installed:
  - `Ansible one -m yum -a "name=demo-tomcat-1 state=absent"`
- Check the latest version of an installed package:
  - `Ansible one -m yum -a "name=demo-tomcat-1 state=latest"`

**Gathering Facts**: Facts can be used for implementing conditional statements in playbook. To find adhoc information of all facts available to Ansible use the following command:
- `Ansible all -m setup`
***

## Playbooks
---
   name: install and configure DB
   hosts: testServer
   become: yes

   vars:
      oracle_db_port_value : 1521

   tasks:
   -name: Install the Oracle DB
      yum: <code to install the DB>

   -name: Ensure the installed service is enabled and running
   service:
      name: <your service name>
- Playbooks are the files where Ansible code is written.
- Playbooks are written in YAML format.
- YAML stands for "Yet Another Markup Language".
- Playbooks contain the steps which the user wants to execute on a particular machine.
- Playbooks run sequentially.
- Each playbook is an aggregation of one or more plays in it.
- The function of a play is to map a set of instructions defined against a particular host.
- **Sample Playbook**:
  ```
  ---
   name: install and configure DB
   hosts: testServer
   become: yes

   vars:
      oracle_db_port_value : 1521

   tasks:
   -name: Install the Oracle DB
      yum: <code to install the DB>

   -name: Ensure the installed service is enabled and running
   service:
      name: <your service name>
  ```

**Different YAML Tags**
- **name**: specifies the name of the Ansible playbook.
- **hosts**: specifies the lists of hosts or host group against which tasks will be run. The tasks can be run on the same machine or on a remote machine. One can run the tasks on multiple machines and hence hosts tag can have a group of hosts' entry as well.
- **vars**: define variables that can be used in a playbook.
- **tasks**: a list of actions to be performed. A tasks field contains the name of the task and internally links to a piece of code called a module: a module that should be executed, and arguments that are required for the module to execute.
***

## Roles

- In Ansible, the role is the primary mechanism for breaking a playbook into multiple files consisting of variables, tasks, files, templates, and modules.
- Each role is basically limited to a particular functionality or desired output, with all the necessary steps to provide that result either within that role itself or in other roles listed as dependencies.
- Roles are not playbooks.
- Roles are small functionality which can be independently used but have to be used within playbooks.
- There is no way to directly execute a role.
- Roles have no explicit setting for which host the role will apply to.
- Top-level playbooks are the bridge holding the hosts from an inventory file to roles that should be applied to those hosts.

### Creating a new role

**Role Structure**: Each role is a directory tree in itself. The role name is the directory name within which the `/roles` directory resides. This is done using the `ansible-galaxy` command:
- `ansible-galaxy -h`
- Command Usage: `ansible-galaxy [delete|import|info|init|install|list|login|remove|search|setup] [--help] [options] ...`

**Creating a Role Directory**:
```
$ ansible-galaxy init --force --offline vivekrole
- vivekrole was created successfully

$ tree vivekrole/
vivekrole/
├── defaults
│   └── main.yml
├── files ├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md ├── tasks
│   └── main.yml
├── templates ├── tests │   ├── inventory
│   └── test.yml
└── vars
    └── main.yml

8 directories, 8 files
```

### Utilizing Roles in Playbooks

The below playbook is named `vivek_orchestrate.yml` with the hosts: `tomcat-node` and uses two roles: `install-tomcat` and `start-tomcat`.
```
---
- hosts: tomcat-node
roles:
   - {role: install-tomcat}
   - {role: start-tomcat}
```

Inside the directories for each role is a tasks directory which contains a file called `main.yml`. The `main.yml` file contents for `install-tomcat` are:
```
---
#Install vivek artifacts
-  
   block:
      - name: Install Tomcat artifacts
         action: >
            yum name = "demo-tomcat-1" state = present
         register: Output

   always:
      - debug:
         msg:
            - "Install Tomcat artifacts task ended with message: {{Output}}"
            - "Installed Tomcat artifacts - {{Output.changed}}"
```

The contents of `main.yml` of `start-tomcat` are:
```
#Start Tomcat          
-  
   block:
      - name: Start Tomcat
      command: <path of tomcat>/bin/startup.sh"
      register: output
      become: true

   always:
      - debug:
         msg:
            - "Start Tomcat task ended with message: {{output}}"
            - "Tomcat started - {{output.changed}}"
```

- The advantage of breaking the playbook into roles is that anyone who wants to use the `intsall-tomcat` feature can call the `install-tomcat` role.
***

## Variables

Variables in Ansible are similar to those in other programming languages and can even have conditions put around them. Below is an example:
```
- hosts : <your hosts>
vars:
tomcat_port : 8080
```

Below is an example using the above variable in the `main.yml` file for the role `install-tomcat`:
```
block:
   - name: Install Tomcat artifacts
      action: >
      yum name = "demo-tomcat-1" state = present
      register: Output

   always:
      - debug:
         msg:
            - "Install Tomcat artifacts task ended with message: {{Output}}"
            - "Installed Tomcat artifacts - {{Output.changed}}"
```
Here are the keywords used in the above code:
- `block`: Ansible syntax to execute a given block
- `name`: Relevant name of the block.
- `action`: The code next to the action tag is the task to be executed.
- `register`: The output of the action is registered using the register keyword and `Output` is the variable name which holds the action output.
- `always`: states that the code following it will always be executed.
- `msg`: displays a message.

When a variable is referenced with `{{variable}}` the value stored in the variable is read. If a variable has sub properties, these can be referenced in dotted decimal notation: `{{variable.subvalue}}`.

### Exception Handling in Playbooks

Below is an example of exception handling in Ansible:
```
tasks:
   - name: Name of the task to be executed
      block:
         - debug: msg = 'Just a debug message , relevant for logging'
         - command: <the command to execute>

      rescue:
         - debug: msg = 'There was an exception.. '
         - command: <Rescue mechanism for the above exception occurred)

      always:
         - debug: msg = "this will execute in all scenarios. Always will get logged"
```
- `rescue` and `always` are the keywords specific to exception handling
- `block` is where the code is written
- If the command written inside the `block` feature fails, then the execution reaches the `rescue` block and it gets executed. In case there is no error in the command under the `block` feature, then `rescue` will not be executed.
- `always` gets executed in all cases.

### Loops

Below is an example in using loops in Ansible:
```
---
#Testing
- hosts: tomcat-node
   tasks:
      - name: Install Apache
      shell: "ls *.war"
      register: output
      args:
         chdir: /opt/ansible/tomcat/demo/webapps

      - file:
         src: '/opt/ansible/tomcat/demo/webapps/{{ item }}'
         dest: '/users/demo/vivek/{{ item }}'
         state: link
      with_items: "{{output.stdout_lines}}"
```
- The task is to copy the set of all the `.war` files from one directory to tomcat webapps folder.
- The `shell` command lists all the `.war` files in the current directory.
- The command output is saved to the `output` variable.
- To loop, the `with_items` syntax is used.
- `with_items: "{{output.stdout_lines}}"`: `output.stdout_lines` gives the line by line output and then the playbook loops on the output with the `with_items` command.
- The `args:` block is used to change directories to the location of the `.war` files. Then the `- file:` block is used to set the source and destination of the files being moved. `{{ item }}` references the data being referenced at the current state of the loop and the `state` command is used to specify that the files should be moved to the `dest` directory.

### Conditionals

Conditionals are used when it is needed to run a specific step based on a condition.
```
---
#Testing
- hosts: all
   vars:
      test1: "Hello Vivek"
   tasks:
      - name: Testing Ansible variable
      debug:
         msg: "Equals"
         when: test1 == "Hello Vivek"
```
- In the above example, `Equals` will be printed as the `test1` variable equals the condition specified in the `when` statement.
- The `when` statement can be used with a logical OR and logical AND condition.
***

- Default location of an inventory: `/etc/ansible/hosts`
  - A different hosts file can be specified with the `-i <path>` option.

- Two default groups:
  1. `all`: contains every host
  2. `ungrouped`: contains all hosts that don't have another group aside from all

- Every host will always belong to at least 2 groups:
  - all and ungrouped
  - all and some other group

- Example of a hosts file: (Can be IP address or FQDN)
```
all:
  hosts:
    192.168.157.131
```

## Variables

- Can be defined in playbooks, in an inventory, in reusable files and roles, or at the command line.
  - Variables can also be created by assigning the return value of a task to a new variable during a playbook run.
  - Variables can be re-used in module arguments, in conditional "when" statements, in templates, and in loops.
  - Variable names can only include letters, numbers, and underscores, and cannot begin with a number. They can begin with an underscore.
  - **Example**: `remote_install_path: /opt/my_app_config`
  - Use Jinja2 syntax to reference a pre-defined variable.
    - Jinja2 variables use double-curly braces.
    - **Example**:
      ```
      ansible.builtin.template:
        src: foo.cfg.j2
        dest: '{{ remote_install_path }}/foo.cfg'
      ```

- Ansible allows Jinja2 loops and conditionals in templates but not in playbooks.

- **List Variables**: combines a variable name with multiple values. The value can be stored as an itemized list or in square brackets [] separated with commas.
  - **Example**:
    ```
    region:
      - northeast
      - southeast
      - midwest
    ```
  - To reference an item in a list variable, place the value position in square brackets after the variable `name: region: "{{ region[0] }}"`

- **Dictionary Variable**: stores data in key value pairs.
  - **Example**:
    ```
    foo:
      field1: one
      field2: two
    ```
  - Reference the values using either bracket notation or dot notation.
    - `foo['field1']`
    - `foo.field1`

- **Registering Variables**: create variables from the output of an Ansible task with the keyword `register`.
  - **Example**:
    ```
    - hosts: web_servers
      tasks:
        - name: Run a shell command and register its output as a variable
          ansible.builtin.shell: /usr/bin/foo
          register: foo_result
          ignore_errors: true

        - name: Run a shell command using the output of the previous task
          ansible.builtin.shell: /usr/bin/foo
          when: foo_result.rc==5
    ```

- **Referencing nested variables**: use bracket or dot notation.
  - **Facts**: data related to remote systems, including operating systems, IP address, attached filesystems, and more. These can be accessed using the `ansible_facts` variable.
  - `{{ ansible_facts["eth0"]["ipv4"]["address"] }}`

- **Transforming variables with Jinja2 filters**: Jinja2 filters let you transform the value of a variable within a template expression.
  - For example, the `capitalize` filter capitalizes any value passed to it.

- Unique values like non-standard SSH ports work well as host variables.
  - You can add them to your Ansible inventory by adding the port number after the hostname with a colon: `badwolf.example.com: 5309`

- **Inventory Aliases**: Used to refer to inventory items by an alias:
  - **Example**:
    ```
    ...
      hosts:
        jumper:
          ansible_port: 5555
          ansible_host: 192.0.2.50
    ```

- **Group Variables**: create variables for an entire group of hosts:
  - **Example**:
    ```
    atlanta:
      hosts:
        host1:
        host2:
      vars:
        ntp_server: ntp.atlanta.example.com
        proxy: proxy.atlanta.example.com
    ```

- **Inheriting variable values: group variables for groups of groups**: groups of groups can be made using the `children:` entry. Variables can be applied to these groups using `vars:`.
  - **Example**:
  ```
  all:
    children:
      usa:
        children:
          southeast:
            children:
              atlanta:
                hosts:
                  host1:
                  host2:
              raleigh:
                hosts:
                  host2:
                  host3:
          vars:
            some_server: foo.souteast.example.com
            halon_system_timeout: 30
            self_destruct_countdown: 60
            escape_pods: 2
          northwest:
          northeast:
          southwest:
  ```
  - Any host that is a member of a child group is automatically a member of the parent group.
  - A child group's variables will have higher precedence (override) a parent group's variables.
  - Groups can have multiple parents and children, but not circular relationships.
  - Hosts can be in multiple groups, but there will only be one instance of a host, merging the data from the multiple groups.

- **Organizing host and group variables**: Host and group variable files must use YAML syntax.
  - Ansible loads host and group variable files by searching paths relative to the invetory file or the playbook file.
  - If your inventory file at `/etc/ansible/hosts` contains a host named `foosball` that belongs to two groups: `raleigh` and `webservers`, that host will use variables in YAML files at the following locations:
    - `/etc/ansible/group_vars/raleigh/vars.yml`
    - `/etc/ansible/group_vars/webservers/vars.yml`
    - `/etc/ansible/host_vars/foosball.yml`
  - You can also create directories named after your groups or hosts. Ansible will read all the files in these directories in lexicographical order.
    - **Example**:
      - `/etc/ansible/group_vars/raleigh/db_settings`
      - `/etc/ansible/group_vars/raleigh/cluster_settings`
    - All hosts in the raleigh group will have the variables defined in these files available to them.
***

## Connecting to hosts: behavioral inventory parameters

- `ansible_connection`: Connction type to the host
  - Can be smart, ssh, or paramiko. Default is smart.

**General for all connections**:
- `ansible_host`: The name of the host to connect to. If different from the alias you wish to give it.
- `ansible_port`: The connection port number (default is 22).
- `ansible_user`: The username to use when connecting to the host.
- `ansible_password`: The password to use to authenticate to the host.

**Specific to the SSH connection**:
- `ansible_ssh_private_key_file`: Private key file used by ssh.
- `ansible_ssh_common_args`: This setting is always appended to the default command line for sftp, scp, and ssh. Useful to configure a Proxy Command for a certain host (or group).
- `ansible_sftp_extra_args`: This setting is always appended to the default sftp command line.
- `ansible_scp_extra_args`: This setting is always appended to the default scp command line.
- `ansible_ssh_extra_args`: This setting is always appended to the default ssh command line.
- `ansible_ssh_pipelining`: Determines whether or not to use SSH pipelining. This can override the ssh_executable setting in `ansible.cfg`.
- `ansible_ssh_executable`: This setting overrides the default behavior to use the system ssh. This can override the `ssh_executable` setting in `ansible.cfg`.

**Privilege Escalation**:
- `ansible_become`: Equivalent to `ansible_sudo` or `ansible_su`, allows to force privilege escalation.
- `ansible_become_method`: Allows to set privilege escalation method.
- `ansible_become_user`: Equivalent to `ansible_sudo_user` or `ansible_su_user`, allows to set the user you become through privilege escalation.
- `ansible_become_password`: Equivalent to `ansible_sudo_password` or `ansible_su_password`, allows you to set the privilege escalation password.
- `ansible_become_exe`: Equivalent to `ansible_sudo_exe` or `ansible_su_exe`, allows you to set the executable for the escalation method selected.
- `ansible_become_flags`: Equivalent to `ansible_sudo_flags` or `ansible_su_flags`, allows you to set the flags passed to the selected escalation method. This can be also set globally in `ansible.cfg` in the `sudo_flags` option.

**Remote host environment parameters**:
- `ansible_shell_type`: The shell type of the target system. You should not use this setting unless you have set the `ansible_shell_executable` to a non-Bourne (`sh`) compatible shell. By default commands are formatted using sh-style syntax. Setting this to `csh` or `fish` will cause commands executed on target systems to follow those shell's syntax instead.
- `ansible_python_interpreter`: The target host python path. This is useful for systems with more than one Python or not located at `/usr/bin/python` such as *BSD, or where `/usr/bin/python` is not a 2.X series Python. We do not use the `/usr/bin/env` mechanism as that requires the remote user's path to be set right and also assumes the python executable is named python, where the executable might be named something like python2.6.
- `ansible_*_interpeter`: Works for anything such as ruby or perl and works just like `ansible_python_interpreter`. This replaces shebang of modules which will run on that host.
- `ansible_shell_executable`: This sets the shell the ansible controller will use on the target machine, overrides executable in `ansible.cfg` which defaults to `/bin/sh`.
***

## Ansible Vault

- Keep the names of variables accessible while encrypting their values:
  1. Create a `group_vars/` subdirectory named after a group.
  2. Inside this subdirectory, create two files named `vars.yml` and `vault.yml`.
  3. In the `vars.yml` file, define all of the variables needed, including any sensitive ones.
  4. Copy all the sensitive variables over to the `vault.yml` file and prefix these variables with `vault_`.
  5. Adjust the variables in the `vars.yml` file to point to the matching `vault_` variables using Jinja2 syntax.
    - `db_password: {{ valut_db_password }}`
  6. Encrypt the vault file to protect its contents.
    - `ansible-vault encrypt vault.yml`
  7. Use the variable name from the `vars.yml` file in your playbooks.

- Example command using `vault.yml` file:
  - `sudo ansible all -m ping --ask-vault-pass`
***

## Playbooks

- Playbooks can declare configurations and orchestrate steps of any manual ordered process, on multiple sets of machines, in a defined order.
  - These tasks can be run synchronously or asynchronously.
  - Playbooks are written in a YAML format.
  - Composed of one or more 'plays' in an ordered list.
  - Each play executes part of the overall goal of the playbook, running one or more tasks. Each task calls an Ansible module.
  - At a minimum, each play defines two things:
    1. The managed nodes to target, using a pattern.
    2. At least one task to execute.

- **Playbook Example**:
```
- name: Update Web Servers
  hosts: webservers
  remote_user: root

  tasks:
    - name: Ensure apache is at the latest version
      ansible.builtin.yum:
        name: httpd
        state: latest
    - name: Write the apache config file
      ansible.builtin.template:
        src: /srv/httpd.j2
        dest: /etc/httpd.conf

    - name: Update db servers
      hosts: databases
      remote_user: root

      tasks:
        - name: Ensure postgresql is at the latest version
          ansible.builtin.yum:
            name: postgresql
            state: latest
        - name: Ensure that postgresql is started
          ansible.builtin.service:
            name: postgresql
            state: started
```

- When you run a playbook, Ansible returns information about connections, the name lines of all your plays and tasks, whether each tasks has succeeded or failed on each machine, and whether each task has made a change on each machine. At the bottom of the playbook execution, Ansible provides a summary of the nodes that were targeted and how they performed. General failures and fatal "unreachable" communication attempts are kept separate in the counts.

- To run a playbook, use the `ansible-playbook` command:
  - `ansible-playbook playbook.yml -f 10`

- `ansible-pull` is a small script that will checkout a repo of configuration instructions from `git`, and then run `ansible-playbook` against that content.

- `ansible-lint` provides `ansible-specific` feedback on a playbook:
  - `ansible-lint verify-apache.yml`

- `ansible-playbook` has commands for validating playbooks: `--check`, `--diff`, `--list-hosts`, `--list-tasks`, and `--syntax-check`.
***

## Enabling SSH Key based authentication (or passwordless SSH)

1. Create an SSH key for your username.
  - `ssh-keygen -q -b 2048 -t rsa -N "" -C "Creating SSH" -f ~/.ssh/id_rsa`
2. Copy the key file to the remote server to which you would like to login.
  - `ssh-copy-id spar@192.168.157.131`
  - This will copy your public key to the remote server and append it to `/home/<user>/.ssh/authorized_keys`.
***

## AWX Ansible (Open-Source Ansible Tower)

We use AWX version 17.0.1 as any version afterwards drops Docker support for Kubernetes.

### Installation

1. Clone into the latest stable release:
  - `git clone -b 17.0.1 https://github.com/ansible/awx.git`
2. Install dependencies:
  - `sudo apt-get update && sudo apt-get install ansible docker docker.io docker-compose pip make git python3 python-setuptools pwgen python3-virtualenv`
  - `sudo pip3 install docker docker-compose`
    - If using a web proxy, set the `HTTP_PROXY` environment variable, (add to the user's `~/.bashrc` file).
    - `HTTP_PROXY=192.168.157.128:3128`
    - `source ~/.bashrc`
    - Or: `sudo pip3 install docker docker-compose --proxy http://192.168.157.128:3128`
3. Create a docker group and add your user:
  - `sudo groupadd docker`
  - `sudo usermod -aG docker $USER`
  - `init 6`
4. Generate a self-signed SSL certificate in the AWX directory if using SSL:
  - `openssl req -newkey rsa:4096 -x509 -sha256 -days 3650 -nodes -out awx.crt -keyout awx.key`
5. Create required directories:
  - `mkdir ~/.awx`
  - `mkdir ~/.awx/pgdocker`
  - `mkdir ~/.awx/awxcompose`
  - `sudo mkdir /opt/my-envs/`
6. Copy over the SSL certificates:
  - `mv awx.crt ~/.awx/`
  - `mv awx.key ~/.awx/`
7. Set the required variables in the AWX inventory file:
  - Located in the installer directory in the github download.
  - `ssl_certificate="~/.awx/awx.crt"`
  - `ssl_certificate_key="~/.awx/awx.key"`
  - `http_proxy=http://192.168.157.128:3128`
  - `https_proxy=http://192.168.157.128:3128`
  - `admin_password=asdfgh3`
8. Run the installer
  - `cd` into the `installer` directory.
  - `ansible-playbook -i inventory install.yml`
