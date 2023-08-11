# DEVOPS and CI/CD Plans and Notes
***

**Infrastructure as Code (IaC)**: the process of using code to describe and manage infrastructure like VMs, network switches, and cloud resources such as Amazon Relational Database Service (RDS). An example of IaC is Vagrant.

**Configuration Management (CM)**: the process of configuring those resources for a specific purpose in a predictable, repeatable manner. An example of CM is Ansible.

## Vagrant Install and Configuration

```
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant
```

**Install Vagrant Plugin for VirtualBox Guest Additions**:
- `vagrant plugin install vagrant-vbguest`

**Vagrantfile Configuration**:
- **Operating System**: vagrant supports many OS base images, called *boxes*, by default.
  - Can be found at: https://app.vagrantup.com/boxes/search/
  - For `ubuntu/focal64`, set the following line at the top of the Vagrant file after the opener: `config.vm.box = "ubuntu/focal64"`
- **Networking**: A VM's network options can be set for different network scenarios, such as *static IP* or *DHCP*.
  - For a private network using DHCP, use the following setting: `config.vm.network "private_network", type: "dhcp"`
- **Providers**: a plug-in that knows how to create and manage a VM.
  - Each provider has common options like CPU, disk, and memory.
  - Vagrant will use the provider's API or command line options to create the VM.
  - Location of providers: https://www.vagrantup.com/docs/providers/
  - Set near the bottom of the file and looks like:
  ```
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.name = "dftd"
    vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
    vb.customize ["modifyvm", :id, --uartmode1", "file", File::NULL]
  end
  ```

**Basic Vagrant Commands**:
- `vagrant up`: Creates a VM using the Vagrantfile as a guide.
- `vagrant destroy`: Destroys the running VM.
- `vagrant status`: Checks the running status of a VM.
- `vagrant ssh`: Accesses the VM over Secure Shell.
***

## Ansible

Ansible uses a *declarative configuration style*, which means it allows someone to describe what the desired state of infrastructure should look like.

Ansible is written in Python and uses YAML (*Yet Another Markup Language*), a data serialization language to describe complex data structures and tasks.

Ansible applies its configuration changes over *Secure Shell (SSH)*.

### Installation

https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

### Key Concepts

- **Playbook**: a collection of ordered tasks or roles that can be used to configure hosts.
- **Control Node**: Any Unix system that has Ansible installed on it. Playbooks and commands are run from a control node.
- **Inventory**: A file that contains a list of hosts or groups of hosts that Ansible can communicate with.
- **Module**: Encapsulates the details of how to perform certain actions across operating systems, such as how to install a software package.
- **Task**: a command or action (such as installing software or adding a user) that is executed on a managed host.
- **Role**: A group of tasks and variables that is organized in a standardized directory structure, defines a particular purpose for the server, and can be shared with other users for a common goal. A typical role could configure a host to be a database server. This role would include all the files and instructions necessary to install the database application, configure user permissions, and apply seed data.

### SSH over Ansible

- Each user's home folder has a file called *authorized_keys*. This file contains a list of public keys the SSH server can use to authenticate that user.
  - To authenticate a user, append their public key to the end of the *authorized_keys* file.

- Use Ansible to copy an SSH public key to an *authorized_keys* file on a remote machine:

```
- name: Set authorized key file from local user
  authorized_key:
    user: bender
    state: present
    key: "{{ lookup('file', lookup('env','HOME') + '/.ssh/dftd.pub') }}"
```

- The contents of the public key are obtained using Ansible evaluation expansion operators `{{ }}` and a built-in Ansible function called `lookup`.
- The `lookup` function retrieves information from outside resources, based on the plug-in specified as its first argument.

#### Adding Two-Factor Authentication

- Two-factor authentication relies on providing two out of these three things:
  1. **Something you know**: password or pin
  2. **Something you have**: phone or hardware authentication device, such as a YubiKey
  3. **Something you are**: fingerprint or voice.
- The Google Authenticator package can be used to configure a VM to use a *time-based one-time password* (TOTP) token for logging in.
  - These TOTP tokens are usually generated from an application like `oathtool` and are valid for only a short period of time.
    - https://www.nongnu.org/oath-toolkit
- A PAM module will also be needed to enforce two-factor authentication on a VM (`libpam-google-authenticator`).

**Steps to Accomplish 2FA** (via Ansible):
1. Install the `libpan-google-authenticator` package.

```
- name: Install the libpam-google-authenticator package
  apt:
    name: "libpam-google-authenticator"
    update_cache: yes
    state: present
```

2. Copy over pre-configured `GoogleAuthenticator` config.

```
- name: Copy over preconfigured GoogleAuthenticator config
  copy:
    src: ../ansible/chapter3/google_authentictor
    dest: /home/bender/.google_authenticator
    owner: bender
    group: bender
    mode: 0600
```

3. Disable password authentication for SSH.

```
- name: Disable password authentication for SSH
  lineinfile:
    dest: "/etc/pam.d/sshd"
    regex: "@include common-auth"
    line: "#@include common-auth"
```

4. Configure PAM to use `GoogleAuthenticator` for SSH logins.

```
- name: Configure PAM to use GoogleAuthenticator for SSH logins
  lineinfile:
    dest: "/etc/pam.d/sshd"
    line: "auth required pam_google_authenticator.so nullok"
```

  - The `nullok` option tells PAM that this setting is optional to prevent locking out users until they have successfully configured 2FA.
5. Set `ChallengeResponseAuthentication` to `Yes`.
  - The `ChallengeResponseAuthentication` option tells SSH to enable a keyboard response prompt when authenticating.

```
- name: Set ChallengeResponseAuthentication to Yes
  lineinfile:
    dest: "/etc/ssh/sshd_config"
    regexp: "^ChallengeResponseAuthentication (yes|no)"
    line: "ChallengeResponseAuthentication yes"
    state: present
```

6. Set Authentication Methods for *bender*, *vagrant*, and *ubuntu*.

```
- name: Set Authentication Methods for bender, vagrant, and ubuntu
  blockinfile:
    path: "/etc/ssh/sshd_config"
    block: |
      Match User "ubuntu,vagrant"
        AuthenticationMethods publickey
      Match User "bender,!vagrant,!ubuntu"
        AuthenticationMethods publickey,keyboard-interactive
    state: present
  notify: "Restart SSH Server"
```

  - The pipe character `|` is YAML notation for introducing a multiline string: the block of text, where the task uses an SSH server configuration option called `Match` that allows the application of certain criteria to specific users.
7. Insert an additional line that reads: Restart SSH Server.
  - The `notify` Ansible option triggers a `handler` to perform a single task.
  - A `handler` is just like any other task, but it's executed only once and has a globally unique name across the whole playbook.

```
- name: Restart SSH Server
  service:
    name: sshd
    state: restarted
```

**Testing SSH Access**:
- `ssh -i ~/.ssh/dftd -p 2222 bender@localhost`
- Every code used during each login is removed from the `/home/bender/.google_authenticator`. To get more, run `vagrant provision` again.
- To generate new tokens on the host itself and add them to the `.google_authenticator` file use the `oathtool` program and the secret at the top of the user's `.google_authenticator` file.
  - `apt install oathtool`
  - `oathtool --totp --base32 "QLIUWM4UVD7E5SI6PPVZ2EGRFU"`

### Controlling User Access with SUDO

#### Install the Greeting Web Application

This is to show how to install an application that will require `sudo` access later on to manipulate for the `developers` group.

1. Install `python3-flask`, `gunnicorn3`, and `nginx`.

```
- name: Install python3-flask, gunicorn3, and nginx
  apt:
    name:
      - python3-flask
      - gunicorn3
      - nginx
    update_cache: yes
```

2. Copy Flask Sample Application

```
- name: Copy Flask Sample Application
  copy:
    src: "../ansible/chapter4/{{ item }}"
    dest: "/opt/engineering/{{ item }}"
  group: developers
  mode: '0750'
  loop:
    - greeting.py
    - wsgi.py
```

3. Copy `Systemd` Unit file for Greeting

```
- name: Copy Systemd Unit file for Greeting
  copy:
    src: "../ansible/chapter4/greeting.service"
    dest: "/etc/systemd/system/greeting.service"
```

**Systemd File**:

```
[Unit]
Description=The Highly Complicated Greeting Application
After=network.target

[Service]
Group=developers
WorkingDirectory=/opt/engineering
ExecStart=/usr/bin/gunicorn3 --bind 0.0.0.0:5000 --access-logfile - --error-logfile - wsgi:app
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

4. Start and enable Greeting Application

```
- name: Start and enable Greeting Application
  systemd:
    name: greeting.service
    daemon_reload: yes
    state: started
    enabled: yes  
```

`sudo` is a program that allows a user to run commands and/or other programs as a different user.

A `sudoers` file is the place where `sudo` security policies are configured (for users and groups) that invoke the `sudo` command.
- Consists of sections called `Defaults`, `User Specifications`, and `Aliases`.
- A `sudoers` file is read from top-down. The last-matching rule always wins.
- The `Defaults` section allows for some `sudoers` options to be overridden at runtime, such as setting environment variables that users have access to when they run `sudo`.
- The `User Specifications` section determines which commands users can run and on which host they can run them.
- The `Aliases` section references other objects inside the file. There are four aliases:
  - `Host_Alias`: A host or a group of hosts
  - `Runas_Alias`: A list of users or groups a command can be run as
  - `Cmnd_Alias`: Specifies a command or multiple commands.
  - `User_Alias`: A user or group of users.

The Ansible module `template` can be used to build a `sudoers` file using the Jinja2 template engine for Python templates. These files are usually stored in the Ansible directory under the `templates/` directory.

```
- set_fact:
  gretting_application_file: "/opt/engineering/greeting.py"
```
  - Creates a variable named `greeting_application_file` and sets its value to `/opt/engineering/greeting.py`.

```
- name: Create sudoers file for the developers group
  template:
    src: "../ansible/templates/developers.j2"
    dest: "/etc/sudoers.d/developers"
    validate: 'visudo -cf %s'
    owner: root
    group: root
    mode: 0440
```

**SUDOERS file template** (`developers.j2`):

```
# Command alias
Cmnd_Alias        START_GREETING    = /bin/systemctl start greeting , \
                                       /bin/systemctl start greeting.service
Cmnd_Alias        STOP_GREETING     = /bin/systemctl stop greeting , \
                                       /bin/systemctl stop greeting.service
Cmnd_Alias        RESTART_GREETING  = /bin/systemctl restart greeting , \
                                       /bin/systemctl restart greeting.service

# Host Alias
Host_Alias LOCAL_VM = {{ hostvars[inventory_hostname]['ansible_default_ipv4']
['address'] }}
# User specification
%developers LOCAL_VM = (root) NOPASSWD: START_GREETING, STOP_GREETING, \
                        RESTART_GREETING, \
                        sudoedit {{ greeting_application_file }}
```

### Automating a Host-Based Firewall with Ansible

- **Uncomplicated Firewall (UFW)**: a software application that provides a thin wrapper around the iptables framework, which is the root of kernel-based packet filtering for Unix OSs.
- UFW has three firewall chains:
  - **Input Chain**: Filters packets destined for the host.
  - **Output Chain**: Filters packets originating from the host.
  - **Forward Chain**: Filters packets that are being routed through the host.
- UFW rules are read top-down, so default drop needs to be last if implemented.

**Ansible Automation for UFW**:
1. Turn `Logging` level to low.

```
- name: Turn Logging level to low
  ufw:
    logging: 'low'
```

2. Allow SSH over port 22.

```
- name: Allow SSH over port 22
  ufw:
    rule: allow
    port: '22'
    proto: tcp
```

3. Allow all access to port 5000.

```
- name: Allow all access to port 5000
  ufw:
    rule: allow
    port: '5000'
    proto: tcp
```

4. Rate limit excessive abuse on port 5000.

```
- name: Rate limit excessive abuse on port 5000
  ufw:
    rule: limit
    port: '5000'
    proto: tcp
```

5. Drop all other traffic.

```
- name: Drop all other traffic
  ufw:
    state: enabled
    policy: deny
    direction: incoming
```

***

## Docker

A *container* is the running instance of an application based off a container image.
- Provides a predictable and isolated way to create and run code.
- Container images package an application and its dependencies into portable artifacts that can be easily distributed and run.

The Docker framework consists of a Docker daemon (server), a docker command line client, and other tools. Docker uses Linux kernel features to build and run containers.
- **OS-Level Virtualization**: partitions the operating system into separate isolated servers.

### Docker Components

**Dockerfile**: describes how to build a container image from an application.
- **Dockerfile Instructions**: Each Dockerfile contains the instructions that teach the Docker server how to turn an application into a container image. Each instruction represents a specific job and creates a new layer inside the container image.
  - **FROM**: Specifies the parent or base image from which to build the new image **(MUST BE THE FIRST COMMAND IN THE FILE)**.
  - **COPY**: Adds files from your current directory (where the Dockerfile resides) to a destination in the image filesystem.
  - **RUN**: Executes a command inside the image.
  - **ADD**: Copies new files or directories from either a source or a URL to a destination in the image filesystem.
  - **ENTRYPOINT**: Makes your container run like an executable (which you can think of as any LInux command line application that takes arguments on your host).
  - **CMD**: Provides a default command or default parameters fro the container (can be used in conjunction with ENTRYPOINT).

**Dockerfile Format**:

```
# Comment
INSTRUCTION arguments
```

Docker runs instructions in a Dockerfile in order. A Dockerfile must begin with a `FROM` instruction. This may be after parser directives, comments, and globally scoped ARGs. The `FROM` instruction specifies the *Parent Image* from which you are building. `FROM` may only be preceded by one or more `ARG` instructions, which declare arguments that are used in `FROM` lines in the Dockerfile.

Environment variables (declared with the `ENV` statement) can also be used in certain instructions as variables to be interpreted by the Dockerfile. Escapes are also handled for including variable-like syntax into a statement literally.

The `${variable_name}` syntax also supports a few of the standard bash modifiers as specified below:
  - `${variable:-word}` incidates that if `variable` is set then the result will be that value. If `variable` is not set then `word` will be the result.
  - `${variable:+word}` indicates that if `variable` is set then `word` will be the result, otherwise the result is the empty string.
In all cases, `word` can be any string, including additional environment variables.

Escaping is possible by adding a `\` before the variable: `\$foo` or `\${foo}`, for example, will translate to `$foo` and `${foo}` literals respectively.

Example (parsed representation is displayed after the `#`):

```
FROM busybox
ENV FOO=/bar
WORKDIR ${FOO}    # WORKDIR /bar
ADD . $FOO        # ADD . /bar
COPY \$FOO /quux  # COPY $FOO /quux
```

Environment variables are supported by the following list of instructions in the `Dockerfile`:
  - `ADD`, `COPY`, `ENV`, `EXPOSE`, `FROM`, `LABEL`, `STOPSIGNAL`, `USER`, `VOLUME`, `WORKDIR`, and `ONBUILD` (when combined with one of the supported instructions above).
Environment variable substitution will use the same value for each variable throughout the entire instruction. For example:

```
ENV abc=hello
ENV abc=bye def=$abc
ENV ghi=$abc
```

will result in `def` having a value of `hello`, not `bye`. However, `ghi` will have a value of `bye` because it is not part of the same instruction that set `abc` to `bye`.

#### FROM

`FROM [--platform=<platform>] <image> [AS <name>]`

OR

`FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]`

OR

`FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]`

The `FROM` instruction initializes a new build stage and sets the *Base Image* for subsequent instructions. As such, a valid Dockerfile must start with a `FROM` instruction. The image can be any valid image.
  - `ARG` is the only instruction that may precede `FROM` in the Dockerfile.
  - `FROM` can appear multiple times within a single Dockerfile to create multiple images or use one build stage as a dependency for another. Simply make a note of the last image ID output by the commit before each new `FROM` instruction. Each `FROM` instruction clears any state created by the previous instructions.
  - Optionally a name can be given to a new build stage by adding `AS name` to the `FROM` instruction. The name can be used in subsequent `FROM` and `COPY --from=<name>` instructions to refer to the image built in this stage.
  - The `tag` or `digest` values are optional. If you omit either of them, the builder assumes a `latest` tag by default. The builder returns an error if it cannot find the `tag` value.

The optional `--platform` flag can be used to specify the platform of the image in the case `FROM` references a multi-platform image. For example, `linux/amd64`, `linux/arm64`, or `windows/amd64`. By default, the target platform of the build request is used. Global build arguments can be used in the value of this flag, for example automatic platform ARGs allow you to force a stage to native build platform (`--platform=$BUILDPLATFORM`), and use it to cross-compile to the target platform inside the stage.

```
ARG CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD /code/run-app

FROM extras:${CODE_VERSION}
CMD /code/run-extras
```

An `ARG` declared before a `FROM` is outside of a build stage, so it can't be used in any instruction after a `FROM`. To use the default value of an `ARG` declared before the first `FROM` use an `ARG` instruction without a value inside of a build stage.

#### RUN

`RUN` has 2 forms:
  - `RUN <command>` (*shell* form, the command is run in a shell, which by default is `/bin/sh -c` on Linux or `cmd /S /C` on Windows).
  - `RUN ["executable", "param1", "param2"]` (exec form)
The `RUN` instruction will execute any commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the Dockerfile.

Layering `RUN` instructions and generating commits conforms to the core concepts of Docker where commits are cheap and containers can be created from any point in an image's history, much like source control.

The *exec* form makes it possible to avoid shell string munging, and to `RUN` commands using a base image that does not contain the specified shell executable.

The default shell for the *shell* form can be changed using the `SHELL` command.

In the *shell* form you can use a `\` (backslash) to continue a single `RUN` instruction onto the next line.

#### RUN --mount

`RUN --mount` allows you to create mounts that processes running as part of the build can access. This can be used to bind files from other parts of the build without copying, accessing build secrets or ssh-agent sockets, or creating cache locations to speed up your build.
- Syntax: `--mount=[type=<TYPE>][,option=<value>[,option=<value>]...]`

**Mount types**

| Type | Description |
| ---- | ----------- |
| `bind` (default) | Bind-mount context directories (read-only). |
| `cache` | Mount a temporary directory to cache directories for compilers and package managers. |
| `secret` | Allow the build container to access secure files such as private keys without baking them into the image. |
| `ssh` | Allow the build container to access SSH keys via SSH agents, with support for passphrases. |

**RUN --mount=type=bind**: This mount type allows binding directories (read-only) in the context or in an image to build a container.

| Option | Description |
| ------ | ----------- |
| `target` | Mount path |
| `source` | Source path in the `from`. Defaults to the root of the `from`. |
| `from` | Build stage or image name for the root of the source. Defaults to the build context. |
| `rw`, `readwrite` | Allow writes on the mount. Written data will be discarded. |

**Container Image**: made of different layers that house an application, its dependencies, and anything else the application needs in order to run.
- Can be distributed and served from a service called a *registry*.

The Dockerfile you build creates a container image. This image is made of different layers that house your application, dependencies, and anything else the application needs so it can run. These layers are like snapshots in time of your application's state, so keeping your Dockerfiles in version control along with your source code makes it easier to build new container images every time your application code changes.
- Each layer, or intermediate image, is created each time an instruction in the Dockerfile is executed.
- Each layher (image) is assigned a unique hash, and all layers are cached by default. This means you can share layers with other images, so if a given layer hasn't changed, you don't need to build it again from scratch.
- Docker can stack these layers on top of each other because it uses the *union filesystem (UFS)*, which allows multiple filesystems to come together and create what looks like a single filesystem. The topmost layer is the *container layer*, which is added when you run the container image. It's the only layer that can be written to. All subsequent layers are read only, by design.

**Containers**: The Docker container is a running instance of a container image. In computer programming terms, you can think of the container image as a *class* and the container as an *instance* of that class. When the container starts, the container layer is created. This write-able layer is where all the changes (like writing, deleting, and modifying existing files) will take place.

**Namespaces and Cgroups**: The container is also roped off from the rest of the Linux host by some boundaries and limited views called *namespaces* and *cgroups*. These are kernel features that limit what a container can see and use on a host. Namespaces restrict global system resources for a container. Without namespaces, a container could have free run of the system. Common kernel namespaces include the following:
- **Process ID (PID)**: Isolates the process IDs
- **Network (net)**: Isolates the network interface stack.
- **UTS**: Isolates the hostname and domain name.
- **Mount (mnt)**: Isolates the mount points.
- **IPC**: Isolates the SysV-style interprocess communication.
- **User**: Isolates the user and group IDs.

You also need to control how much memory, CPU, and other physical resourdces a container uses. That's where cgroups come in. Cgroups manage and measure the resources a container can use. They allow you to set resource limitations and prioritization for processes. The most common resources Docker sets with cgroups are memory, CPU, disk I/O, and network. Cgroups make it possible to stop a container from using up all the resources on a host.

### Installing and Testing Docker

*Minikube*, an app that contains the Docker engine and also provides a Kubernetes cluster.
- https://minikube.sigs.k8s.io/

**Start**: `minikube start --driver=virtualbox`
- May need to whitelist the IP that `minikube` is trying to use.

**Installing Docker**: https://docs.docker.com/engine/install/binaries/

- Set your Docker environment variables for `minikube`: `eval $(minikube -p minikube docker-env)`
  - The Docker host environment variables should be exported in your current terminal session.

**Building Docker Containers**:
- Example: `docker build -t dftd/telnet-server:v1 .`
- The `-t` option sets the name and tag for the image.
- Using Git commit hashes as tags is a common practice as each hash is unique and can mark the image's source code version.
- If you omit the tag, Docker uses the latest word as the default tag.

**Verifying Docker Images**: `docker image ls dftd/telnet-server:v1`

**Running the Container**: `docker run -p 2323:2323 -d --name telnet-server dftd/telnet-server:v1`
- The `-p` (port) flag exposes port 2323 outside the container. The left side of the colon is the host port, and the right side is the container port.
- The `-d` (detach) flag launches the container in the background. If you don't supply the `-d` flag the container will run in the foreground of the terminal from which it was launched.
- The `--name` flag sets the container name to the `telnet-server`. Docker, by default, assigns randomly generated names for containers if you don't set them.
- The last argument is the image name, complete with path and tag, from the build step.
- If successful, the `docker run` command will return the *container ID* of the running container and no errors.
- The `-v` (volume flag) can mount a local directory or local file inside the running container. This can be used to share data between a host and a container.
- Check on the container: `docker container ls -f name=telnet-server`

**Stop the container**: `docker container stop telnet-server`

### Other Docker Client Commands

`exec`: The exec command allows you to run a command inside a container or interact with a container as if you were logged into a terminal session.
- `docker exec telnet-server env`
- To run an interactive shell inside the container, use the `docker exec -it <container name> /bin/sh` command.

`rm`: The rm command removes a stopped container.
- `docker container rm telnet-server`
- Use the `-f` flag to force remove a running container.

`inspect`: The inspect docker command returns low-level information about some Docker objects.
- `docker inspect telnet-server`

`history`: The history command displays a container image's history. Useful for viewing the number and sizes of an image's layers.
- `docker history dftd/telnet-server:v1`

`stats`: The stats command displays a real-time update on the resources a container is using. It gathers this information from the cgroups and behaves similarly to the Linux `top` command.
- Use the `--no-stream` flag to take a snapshot of the resources and exit immediately.
- `docker stats --no-stream telnet-server`

### Testing the Container

To connect to the server, pass telnet the hostname or IP address of the server plus the port to which you want to connect. Since the Docker server is running inside a VM (minikube), you'll need the IP address minikube exposes to your local host.
- `minikube ip`

Connect to the telnet server: `telnet <ip address> 2323`

### Getting Logs from the Container

To see all the logs for a container, which is normally logging to STDOUT, use the following command:
- `docker logs <container name>`
***

## Kubernetes

**Orchestration**: refers to managing thousands of containers on different hosts, network ports, and shared volumes. *Kubernetes*, or *K8s*, is the open-source orchestration system many companies use to manage their containers.

A Kubernetes cluster consists of one or more control plane nodes and one or more worker nodes.
- **Node**: can be anything from a cloud VM to a bare-metal racked server to a Raspberry Pi.
- **Control Plane Nodes**: handle things like the Kubernetes API calls, the cluster state, and the scheduling of containers.
  - The core services (such as the API, etcd, and the scheduler) run on the control plane.  
- **Worker Nodes**: run the containers and resources that are scheduled by the control plane.

When networking containers, you must consider all the ports and access they need. Containers can communicate with each other, both inside and outside the cluster.

**Node Affinity**: You can tune a worker node for a specific use case, like high connections, and then create rules to ensure that the applications that need that feature end up on that specific worker node.

### Kubernetes Workload Resources

**Resource**: a type of object that encapsulates state and intent. Kubernetes will maintain a given count for all specified resources.
- Kubernetes resources are defined in a file called a *manifest*.

**Pods**: the smallest building blocks in Kubernetes.
- A Pod is made up of one ore more containers that share network and storage resources.
- Each container can connect to the other containers, and all containers can share a directory between them by a mounted volume.
- Pods aren't deployed directly and are instead incorporated into a higher-level abstraction layer.

**ReplicaSet**: used to maintain a fixed number of identical Pods.
- If a Pod is killed or deleted, the ReplicaSet will create another Pod to take its place.
- ReplicaSets should only be used to create a custom orchestration behavior.

**Deployments**: a resource that manages Pods and ReplicaSets.
- A Deployment's main job is to maintain the state that is configured in its manifest.
- For example, you can define the number of Pods (which are called *replicas* in this context), along with the strategy for deploying new Pods.
- The Deployment resource controls a Pod's lifecycle, from creation, to updates, to scaling, to deletion.
- You can also roll back to earlier versions of a Deployment as needed.

**StatefulSets**: a resource for managing stateful applications, such as PostgreSQL, ElasticSearch, and etcd.
- Similar to a Deployment, it can manage the state of Pods defined in a manifest. However, it also adds features like managing unique Pod names, managing Pod creation, and ordering termination.
- Each Pod in a StatefulSet has its own state and data bound to it.
- If you are adding a stateful application to your cluster, choose a StatefulSet over a Deployment.

**Services**: allow you to expose applications running in a Pod or group of Pods within the Kubernetes cluster or over the internet. You can choose from the following basic Service types:
- **ClusterIP**: This is the default type when you create a Service. It is assigned an internal routable IP address that proxies connections to one or more Pods. You can access a ClusterIP only from within the Kubernetes cluster.
- **Headless**: This does not create a single-service IP address. It is not load balanced.
- **NodePort**: This exposes the Service on the node's IP addresses and port.
- **LoadBalancer**: This exposes the Service externally. It does this either by using a cloud provider's component, like AWS's Elastic Load Balancing (ELB), or a bare-metal solution, like MetalLB.
- **ExternalName**: This maps a Service to the contents of the `externalName` field to a `CNAME` record with its value.
- **NOTE**: Only the `LoadBalancer` and `NodePort` Services can expose a Service outside the Kubernetes cluster.

**Volumes**: basically a directory, or a file, that all containers in a Pod can access, with some caveats.
- Volumes provide a way for containers to share and store data between them.
- If a container in a Pod is killed, the Volume and its data will survive; if the entire Pod is killed, the Volume and its contents will be removed.
- *Persistent Volume (PV)*: a resource in a cluster just like a node that is not linked to a Pod's lifecycle. Pods can use the PV resource, but the PV does not terminate when the Pod does.

**Secrets**: convenient resources for safely and reliably sharing sensitive information (such as passwords, tokens, SSH keys, and API keys) with Pods.
- You can access Secrets either via environment variables or as a Volume mount inside a Pod.
- Secrets are stored in a RAM-backed filesystem on the Kubernetes nodes until a Pod requets them.
- When not used by a Pod, they are stored in memory, instead of in a file on disk.
- Be careful because the Secrets manifest expects the data to be in Base64 encoding, which is not a form of encryption.

**ConfigMaps**: allow you to mount nonsensitive configuration files inside a container.
- A Pod's containers can access the ConfigMap from an environment variable, from command line arguments, or as a file in a Volume mount.
- If your application has a configuration file, putting it into a ConfigMap manifest provides two main benefits.
  - First, you can update or deploy a new manifest file without having to redeploy your whole application.
  - Second, if you have an application that watches for changes in a configuration file, then when it gets updated, your application will be able to reload the configuration without having to restart.

**Namespaces**: allows you to divide a Kubernetes cluster into several smaller virtual clusters.
- If you don't specify a Namespace when creating a resource, it will inherit the Namespace *default*.

### Example Deployment

The most direct way to interact with the cluster is to use the `kubectl` command line application.
- Binary can be downloaded from https://kubernetes.io/docs/tasks/tools/install-kubectl/
- Minikube will also fetch the `kubectl` binary for you the first time you invoke the `minikube kubectl`.
  - When using `minikube kubectl`, most commands will require double dashes (--) between `minikube kubectl` and subcommands. The standalone version of `kubectl`, however, does not need dashes between the commands.

Use the `cluster-info` subcommand to verify that the cluster is up and running: `minikube kubectl cluster-info`

Kubernetes manifests can either be in JSON or YAML. You'll usually find the files co-residing with the application they describe.

To describe a complex object, you'll need multiple fields, subfields, and values to define how a resource behaves.
- Among all these fields and values, there is a subset of required fileds called *top-level fields*. The four top-level fields are as follows:
  - `apiVersion`: This is a Kubernetes API version and group, like apps/v1. Kubernetes uses versioned APIs and groups to deliver different versions of features and support for resources.
  - `kind`: This is the type of resource you want to create, such as a Deployment.
  - `metadata`: This is where you set things like names, annotations, and labels.
  - `spec`: This is where you set the desired behavior for the resource (kind).
- Each of these top-level fields contains multiple subfields. The subfields contain information such as name, replica count, template, and container image.
- **Labels**: provide a way for the user to tag a resource with identifiable key values. For example, you could add a label to all resources that are in the `production environment`.
```
metadata:
  labels:
    environment: production
```

The `explain` sub-command describes the fields associated with each resource type. You can use the dot (.) notation as a type of field separator when searching for nested fields.
- For example, to learn more about a Deployment's `metadata labels` subfield, enter the following in a terminal: `minikube kubectl -- explain deployment.metadata.labels`

To create and update resources, you can pass `minikube kubectl` two sub-commands: `create` and `apply`.
- The `create` subcommand is *imperative*, which means it makes the resource reassemble the manifest file. It also throws an error if the resource already exists.
- The `apply` subcommand is *declarative*, which means it creates the resource if it does not exist and updates it if if does.
- The `-f` flag to instruct `kubectl` to run the operation against all the files in the `kubernetes/` directory. The `-f` flag can take filenames in lieu of directories as well.
  - `minikube kubectl -- apply -f kubernetes/`

Kubernetes provides multiple ways to view any object's status. The easiest method is to use the `minikube kubectl -- get <resource> <name>` command.
- `minikube kubectl -- get deployments.apps telnet-server`

Because you could have hundreds of Pods, you want to narrow down your results with the `-l` label filter flag.
- Enter the following to show only the `telnet-server` Pods: `minikube kubectl -- get pods -l app=telnet-server`

**Scale**: means changing the number of replicas that the Deployment has up and down from the command line, in the case of a change in load.
- The first way to scale is to edit the *deployment.yml* manifest file and apply the changes to the cluster using the `minikube apply` command.
  - `minikube kubectl -- apply -f kubernetes/deployment.yaml`
- The second way is to use the `minikube kubectl scale` command.
  - `minikube kubectl -- scale deployment telnet-server --replicas=3`
  - Verify: `minikube kubectl -- get deployments.apps telnet-server`

In a terminal, enter the following to create a network tunnel to the `telnet-server` Service:
- `minikube tunnel`
- When you want to close the tunnel, press `CTRL-C` to shut it down.
- Get the new external IP address for the `LoadBalancer` Service: `minikube kubectl -- get services telnet-server`

To make sure your Service is wired up to your Pods, one way to check this connection is with the `kubectl get endpoints` command, which will tell you if the Service can find the Pods you specified in the Service `spec.selector` field located in the *service.yaml* file.
- `minikube kubectl -- get endpoints -l app=telnet-server`
- The `ENDPOINTS` column shows the internal Pod IP addresses with ports.
- If your `ENDPOINTS` column has <none>, check that the `spec.selector` field in your Service matches what is in the `spec.template.metadata.labels` field in the *deployment.yaml* file.

Enter the following command to delete a Pod: `minikube kubectl -- delete pod <Pod Name>`
- Get the Pod names with: `minikube kubectl -- get pods -l app=<app name>`

Access the `telnet-server` application logs: `minikube kubectl -- logs <pod name>`
- Enter the following command to fetch all the logs for each Pod: `minikube kubectl -- logs -l app=telnet-server --all-containers=true --prefix=true`
  - Fetch only Pods with this label: `-l app=telnet-server`
  - When you have multiple Pods and want to see all the logs: `--all-containers=true`
  - Each log line with the Pod name from which the log came: `--prefix=true`
***

## Deploying Code

**Continuous Integration/Continuous Deployment (CI/CD)**: the process of getting code from your editor to your stakeholders in a consistent and automated manner.

Continuous integration and continuous deployment are software development methodologies that describe the way code is built, tested, and delivered. The CI steps cover the testing and building of code and configuration changes, while the CD steps automate the deployment (or delivery) of new code.
- During the CI stage, a software engineer introduces new features or bug fixes through a version control system like Git. This code gets run through a series of builds and tests before finally producing an artifact like a container image.
  - This process solves the "works on my machine" problem because everything is tested and built in the same way to produce a consistent product.
- The testing step usually consists of unit tests, integration tests, and security scans.
- The unit and integration tests make sure the application behaves in an expected manner, whether in isolation or interacting with other components in your stack.
- The security scans usually check for known vulnerabilities in your applications software dependencies or for vulnerable base container images you are importing.
- After the testing steps, the new artifact is built and pushed to a shared repository, where the CD stage has access to it.
- During the CD stage, an artifact is taken from a repository and then deployed, usually to production infrastructure.

**Deployment Strategies**

| Name | Description |
| ---- | ----------- |
| Canary | This strategy rolls out new code so only a small subset of users can access it. If the canary's code presents zero errors, the new code can be rolled out further to more customers. |
| Blue-Green | In this strategy, a production service (blue) takes traffic while the new service (green) is tested. If the green code is operating as expected, the green service will replace the blue service, and all customer requests will funnel through it. |
| Rolling | This strategy deploys new codes one by one, alongside the current code in production, until it is fully released. |

### Example Pipeline

The first tool used is Skaffold, and it helps with continuous development for Kubernetes-native applications.
- https://skaffold.dev/docs/install/

The other tool, `container-structure-test`, is a command line application that validates the container image's structure after it's built. It can test whether the image was constructed properly by verifying whether a specific file exists, or it can execute a command and validate its output. You can also use it to verify that a container image was built with the correct metadata, like the ports or environment variables you would set in a Dockerfile.
- https://github.com/GoogleContainerTools/container-structure-test/

The `skaffold.yaml` file describes how to build, test, and deploy your application. This file should live in the root of your project and be kept under version control. The YAML file has three key sections: `build`, `test`, and `deploy`.
- `build`: describes how to build your container image.
- `test`: describes what tests to perform.
- `deploy`: describes how to release your application to the Kubernetes cluster.

You can execute the `skaffold` command with either the `run` or the `dev` subcommand. The `run` subcommand is a one-off that builds, tests, and deploys the application and then exits. It does not watch for any new code changes. The `dev` command does everything `run` does, but it watches source files for any changes. Once it detects a change, it kicks off the `build`, `test`, and `deploy` steps described in the `skaffold.yaml` file.
- After the `dev` subcommand is run successfully, it will wait and block looking for any changes. By default, you'll need to press `CTRL-C` to exit the `skaffold dev` mode. However, when you use `CTRL-C` to exit, the default behavior is to clean up after itself by removing the Deployment and Services from the Kubernetes cluster.
- Add the `--cleanup=false` flag to the end of the `dev` command to bypass this behavior.
  - `skaffold dev --cleanup=false`

**Testing a Rollback**: First, check the rollout history. Every time you deploy new code, Kubernetes tracks the Deployments and saves the resource state at that given time.
- Enter the following in a terminal to fetch the Deployment history for `telnet-server`: `minikube kubectl -- rollout history deployment telnet-server`
- Using the `--record` flag when running `kubectl apply` makes Kubernetes record which command triggered the `deploy`.
- `minikube kubectl -- rollout undo deployment telnet-server --to-revision=1`
- The default is to roll back to the previous revision.
- Verify by running the `minikube kubectl -- get pods` command.
***

## Various

### Generating Passwords

**Install necessary programs**:
- `sudo apt update`
- `sudo apt install pwgen whois`

**Generate password**:
- `pass=$(pwgen --secure --capitalize --numerals --symbols 12 1)`
- `echo $pass | mkpasswd --stdin --method=sha-512; echo $pass`

### Generating a Public Key Pair for SSH

- `ssh-keygen` is the tool used to generate and manage public key pairs on a local host.
  - `ssh-keygen -t rsa -f ~/.ssh/dftd -C dftd`
  - Requires the entry and re-entry of a passphrase.
  - `-C`: specifies the name of the key pair, default is *id_rsa*.
