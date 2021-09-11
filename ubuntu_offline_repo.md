## Offline Repository

https://www.linuxtechi.com/setup-local-apt-repository-server-ubuntu/
https://linuxconfig.org/how-to-create-a-ubuntu-repository-server
https://help.ubuntu.com/community/AptGet/Offline/Repository
https://gist.github.com/jeanlescure/084dd6113931ea5a0fd9

The Ubuntu software repository is organized into four parts:
1. **Main**: Officially supported software
2. **Restricted**: Supported software that is not available under a completely free license
3. **Universe**: Community-maintained, i.e. not officially supported software
4. **Multiverse**: Software that is not free.

The standard Ubuntu installation is a subset of software available from the main and restricted sections.

To determine what suite of your Ubuntu box, run the `lsb_release -a` command. The suite will be the `Codename`.
- For example: `Codename:   focal`

### Steps to create an Offline Ubuntu Repository (The Hard Way)

1. Go to http://archive.ubuntu.com/ubuntu/dists/ and then navigate to the directory that matches the suite of your Ubuntu build.
2. Download the files:
  - `Release`
  - `Release.gpg`
  - The contents files for your architecture (i.e. arm, i386, etc.)
3. Go into each repository and architecture folder and download the following files:
  - `Packages.bz2`
  - `Packages.gz`
  - `Release`
4. Store the files in the same format as the primary repository server. Assuming the files are hosted via a web server, the would be stored like:
  - `/var/www/html/repository/dists/focal/`: Should contain `Release`, `Release.gpg`, and `Contents-amd64.gz`.
  - `/var/www/html/repository/dists/focal/main/binary-amd64/`: Should contain `Release`, `Packages.gz`, and `Packages.xz`.
  - `/var/www/html/repository/dists/focal/main/source/`: Should contain `Release`, `Sources.gz`, and `Sources.xz`.
  - `/var/www/html/repository/dists/focal/restricted/binary-amd64/`: Should contain `Release`, `Packages.gz`, and `Packages.xz`.
  - `/var/www/html/repository/dists/focal/restricted/source/`: Should contain `Release`, `Sources.gz`, and `Sources.xz`.
  - `/var/www/html/repository/dists/focal/multiverse/binary-amd64/`: Should contain `Release`, `Packages.gz`, and `Packages.xz`.
  - `/var/www/html/repository/dists/focal/multiverse/source/`: Should contain `Release`, `Sources.gz`, and `Sources.xz`.
  - `/var/www/html/repository/dists/focal/universe/binary-amd64/`: Should contain `Release`, `Packages.gz`, and `Packages.xz`.
  - `/var/www/html/repository/dists/focal/universe/source/`: Should contain `Release`, `Sources.gz`, and `Sources.xz`.
5. Add the local repository in a host's sources list:
  - Go to **Applications** -> **Add/Remove** to add the offline repository:
  - `deb html://repository/ focal main restricted universe multiverse`

### Steps to create an Offline Ubuntu Repository (The Easy Way)

1. Install and enable apache2:
  - `sudo apt install apache2`
  - `sudo systemctl enable apache2`
2. Create a folder to host the files and change the ownership:
  - `sudo mkdir -p /var/www/html/ubuntu`
  - `sudo chown www-data:www-data /var/www/html/ubuntu`
3. The primary tool that allows the creation of a local repository is `apt-mirror`, install it:
  - `sudo apt install apt-mirror`
4. Grab the latest fork of `apt-mirror` that's actually maintained:
  - `https://github.com/Stifler6996/apt-mirror`
5. Backup the configuration file for `apt-mirror`:
  - `sudo cp /etc/apt/mirror.list /etc/apt/mirror.list.org`
6. Edit the configuration file to change the `set base_path` line to the new location: `/var/www/html/ubuntu`. Also you will need to modify the `deb` lines by changing the default suite (usually artful) to the one you are using. Finally, add lines to use a web proxy if it's required:
  - `vim /etc/apt/mirror.list`
  - `set base_path  /var/www/html/ubuntu`
  - Example of a `deb` line: `deb http://archive.ubuntu.com/ubuntu focal main restricted universe multiverse`
  - `set use_proxy on`
  - `set http_proxy 192.168.157.128:3128`
  - `set proxy_user user`
  - `set proxy_password password`
7. Create a `var/` directory in the repo location and copy over the `postmirror.sh` script:
  - `sudo mkdir -p /var/www/html/ubuntu/var`
  - `sudo cp /var/spool/apt-mirror/var/postmirror.sh /var/www/html/ubuntu/var/`
8. Create `mirror/` and `skel` directories in `/var/www/html/ubunut/` and copy over the `clean.sh` script:
  - `sudo mkdir -p /var/www/html/ubuntu/mirror`
  - `sudo mkdir -p /var/www/html/ubuntu/skel`
  - `sudo cp /var/spool/apt-mirror/var/clean.sh /var/www/html/ubuntu/var/clean.sh`
9. Run the `apt-mirror` command. **NOTE** This takes a long time.
  - `sudo apt-mirror`
10. In Ubuntu 20.04 LTS, `apt-mirror` does sync CNF directory and its files, so we have to manually download and copy the folder and its files. The script below will do this:
  - `vim cnf.sh`
  ```
  #!/bin/bash
  for p in "${1:-focal}"{,-{security,updates}}\
  /{main,restricted,universe,multiverse};do >&2 echo "${p}"
  wget -q -c -r -np -R "index.html*"\
  "http://archive.ubuntu.com/ubuntu/dists/${p}/cnf/Commands-amd64.xz"
  wget -q -c -r -np -R "index.html*"\
  "http://archive.ubuntu.com/ubuntu/dists/${p}/cnf/Commands-i386.xz"
  done
  ```
  - `chmod +x cnf.sh`
  - `bash cnf.sh`
11. This script will create a folder with the name `archive.ubuntu.com` in the present working directory. Copy this folder to the mirror folder:
  - `sudo cp -av archive.ubuntu.com  /var/www/html/ubuntu/mirror/`
12. In case the firewall is running, allow port 80:
  - `sudo ufw allow 80`
13. On the Ubuntu client that will use the offline repository, edit it's `/etc/apt/sources.list` file and make the following changes:
  - `sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak`
  - `sudo nano /etc/apt/sources.list`
  - Comment out the lines referencing Ubuntu repositories and add the following lines:
    - `deb [trusted=yes] http://192.168.157.129/ubuntu/mirror/archive.ubuntu.com/ubuntu focal main restricted universe multiverse`
    - `deb [trusted=yes] http://192.168.157.129/ubuntu/mirror/archive.ubuntu.com/ubuntu focal-updates main restricted universe multiverse`
    - `deb [trusted=yes] http://192.168.157.129/ubuntu/mirror/archive.ubuntu.com/ubuntu focal-security main restricted universe multiverse`
14. Save the file and perform an update:
  - `sudo apt update`
