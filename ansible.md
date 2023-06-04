# Ansible Notes

## Ansible Overview and Setup

The machine in which you install the Ansible program is known as the **Ansible control node**. It will have your Ansible playbooks and other configurations.

The machines or devices that you want to automate are known as **managed hosts**. You will run the Ansible jobs and playbooks from the control node and the jobs will be executed on the target nodes or managed nodes.

Ansible uses default connection methods to communicate with managed nodes, such as `ssh`, `WindRM`, `http`, or other appropriate protocols. 

The Ansible inventory is a file or script that will provide the details about the managed nodes, including the hostname, connection methods, credentials to use, and many other methods.

Ansible plugins are small pieces of code that help to enable a flexible and expandable architecture. For example, there are **connection plugins**, **action plugins**, **become plugins**, and **inventory plugins**.

An Ansible module is a piece of reusable and standalone script that can be used to achieve some specific tasks. Modules provide a defined interface with options to accept  arguments and return information to Ansible in JSON format. When you execute a task using a module, the module script will be executed on the target machine using Python or using PowerShell for Windows machines.
- Ansible modules are separated from the Ansible base and distributed as **Ansible content collections**, or simply **Ansible collections**.
- In Ansible, most modules follow a feature called **idempotency**, which means that if the result of performing an action is the same as the current state, then no further action is required. 

Ansible playbooks are files written in YAML format with the instruction list of automation tasks.

The `ansible` command can be used to execute single jobs on managed nodes without a playbook; this is called an **Ansible ad hoc** command.

Use the `ansible-doc` command to lookup modules available to your current ansible installation.
- `ansible-doc -l`: display all modules available to your current ansible environment.
  - Press `/` followed by the name of the module you're looking for to search for the module in the list of available modules.
- `ansible-doc -s dnf`: will display information about modules related to the `dnf` command.
- `ansible-doc dnf`: will display information about the `dnf` ansible module itself.
- `ansible-doc -t apt -l`: will list all the modules dealing with the `apt` program.
- Press `n` while listing out modules in `ansible-doc` to move to the next module in the list.

### Installing Ansible

**RHEL Distros**: `sudo dnf install ansible`

**Debian Distros**: `sudo apt install ansible`

### Deploying Ansible

Configure ansible for your environment, with the `ansible.cfg` looked for by Ansible in the following four places:
- `$ANSIBLE_CONFIG`: Configuration file path in an environment variable
- `./ansible.cfg`: Configureation file in the current directory
- `~/.ansible.cfg`: Configuration file in the home directory
- `/etc/ansible/ansible.cfg`: Default configuration file

In an Ansible inventory file, you can have the hostname (FQDN) as the first entry on a line followed by the `ansible_host` parameter to hold its IP address in the event DNS isn't working.
- Do not use spaces, only hyphens or underscores when naming hosts and host groups.


### Configuring Managed Nodes

Steps to set up SSH key based access on each node:
1. Create a dedicated user on the target node for running ansible commands: 
  - `sudo useradd ansible -s /bin/bash -U -G sudo`
  - `sudo passwd ansible`
  - `sudo mkdir /home/ansible`
  - `sudo mkdir /home/ansible/.ssh`
  - `sudo chown ansible:ansible -R /home/ansible`

2. Enable `sudo` access for the new user for become purposes:
  - Add to the `sudo` group: `sudo usermod -G sudo -a ansible`
  - Or, use `sudoedit` to add the following line to the `/etc/sudoers` file:
    - `ansible ALL=(ALL) NOPASSWD: ALL`
3. Create an SSH key pair on the **Ansible control node**: `ssh-keygen -t rsa -b 4096`
4. Copy the SSH public key from the Ansible control node to managed nodes under the ansible user using the `ssh-copy-id` command: `ssh-copy-id -i ~/.ssh/id_rsa ansible@<hostname>`
5. Add the managed nodes to the allowed hosts for the ansible control node for ssh host key checking:
  - `ssh-keygen -R <hostname>`: Done to ensure that the old entries for a host are removed.
  - `ssh-keyscan -t ecdsa,ed25519 -H <host> >> ~/.ssh/known_hosts 2>&1`

### Configure VIM for Ansible

1. Open a file using vim: `vim <filename>`
2. Press `Esc` followed by `:` and type `set nu` to enable line numbers.

The below table expounds on some `vim` variables:

| Variable | Description | Example |
| -------- | ----------- | ------- |
| ts | tabstop is the number of spaces that a <Tab> in the file counts for. | tabstop =2 |
| et | expandtab uses the appropriate number of spaces to insert a <Tab>. | et |
| sw | shiftwidth is the number of spaces to use for each step of (auto) indent. | shiftwidth=2 |
| sts | softtabstop is the number of spaces that a <Tab> counts for while performing editing operations, such as inserting a <Tab> or using <BS>. | softtabstop=2 |
| nu | number shows the line number in the file. | nu |

Example `.vimrc` file:

```
colorscheme delek
set number
syntax on
set tabstop=2
set cursorline
set cursorcolumn
set expandtab
set shiftwidth=2
set softtabstop=2
```

***

## Ansible Facts, Roles, and Automating Daily Tasks

Ansible and `ansible_facts` can be used to create and update your system inventory database or your own **configuration management database** (CMDB). 
- `ansible_facts` provides detailed and invormative data that's gathered from the target nodes and stored in JSON format. 
- `ansible_facts` can be used to make decisions inside the playbook, such as skipping the task if the system memory is less than the required value or installing specific packages, depending on the operating system version of the target node.

An Ansible role is a collection of tasks, handlers, templates, and variables for configuring the target system so that it meets the desired state.
- The content of Ansible roles is arranged in an organized way. The top-level directory defines the name of the role itself.
- Below is the basic folder structure and default files for an Ansible role:
  - `defaults/main.yml`: This contains variables for the role that can be overwritten when the role is used.
  - `tasks/main.yml`: This contains the main list of tasks to be executed when using the role.
  - `vars/main.yml`: This contains internal variables for the role.
  - `files`: This contains statis files that can be referenced from this role.
  - `templates`: These are the jinja2 templates that can be used via this role.
  - `handlers/main.yml`: This contains the handler definitions.
  - `meta/main.yml`: This defines some metadata for this role, such as the author, license, platform, and dependencies.
  - `tests`: This is an inventory. `test.yml` is a file that can be used to test this role.

**Jinja2** is an extensive templating engine that dynamically generates content using variables and facts. It is possible to create any kind of complex template file that contains variables, loops, and other controls. Inside a playbook, use the `template` module (or the Ansible `template` filter) to convert the Jinja2 template into actual content or a file. Ansible will replace the variable with values as needed.
- For example, below is a Jinja2 template of the `/etc/motd` file:

```
Welcome to {{ ansible_facts.hostname }}
(IP Address: {{ ansible_facts.default_ipv4.address }})

Access is restricted; if you are not authorized to use it
please logout from this system

If you have any issues, please contact {{ system_admin_email }}.
Phone: {{ system_admin_phone | default('1800 1111 2222') }}
```

To deploy the above Jinja2 template in a playbook, use the `template` module similar to the one below:

```
tasks:
  - name: Deploy motd
  template:
    dest: /etc/motd
    src: motd.j2
```

You can deploy the default directory structure for a new role using the `ansible-galaxy` command. For example: `ansible-galaxy role init <role-directory>`

### Collecting System Information and System Scanning

`ansible_facts` contains a lot of information about nested dictionaries and lists. To see the content of `ansible_facts`, run the ad hoc command: `ansible <hostname> -m setup | less`

When there are multiple tasks in a playbook or role, they can be split into multiple files and called using the `include_tasks` module dynamically.

`extra-vars` contains the variables that will override all other variables. Using a dynamic extra variable will help you control the playbook based on the values and is also useful when you are using survey forms in Ansible AWX or Ansible Automation Controller, where variables can be defined in the GUI method (survey forms). `--extra-vars` can be passed as a single value, multiple key-value pairs, in JSON format, or read from a varialbe file, as follows:
- `ansible-playbook site.yml --extra-vars "version=1.23.45 other_variable=foo"`
- `ansible-playbook site.yml --extra-vars '{"version":"1.23.45","other_variable":"foo"}'`
- `ansible-playbook site.yml --extra-vars "@vars_file.json"`

***

## Ansible Vault

**Ansible Vault** is a built-in feature in Ansible to store and manage encrypted content. 

Ansible Vault will encrypt files with a password (vault secret) provided and make sensitive data unreadable to normal users. When Ansible wants to read the data, Ansible will ask for the vault password which can be provided via keyboard or through a secret file.

To create a valut file with the following command: `ansible-vault create vars/secrets`
- Once this has been done, a text editor will open so that you can enter the content of your sensative file. 
- To view the file's contents, use the `ansible-vault view vars/secrets` command and enter the vault password.