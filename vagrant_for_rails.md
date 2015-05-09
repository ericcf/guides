# How to set up a Rails app development environment on OSX using Vagrant

These instructions have been tested on 10.10.3 (Yosemite).

## Install VirtualBox

Download it here: http://dlc-cdn.sun.com/virtualbox/4.3.26/VirtualBox-4.3.26-98988-OSX.dmg

Install VirtualBox and start the application

Download the extension pack here: http://dlc-cdn.sun.com/virtualbox/4.3.26/Oracle_VM_VirtualBox_Extension_Pack-4.3.26-98988.vbox-extpack

Double click the extension pack to install it

## Install Vagrant

Download it here: https://dl.bintray.com/mitchellh/vagrant/vagrant_1.7.2.dmg

## Set up CentOS

Change to the directory where you'll store the configuration for your VM

Download a CentOS 6.5 Vagrant box (this may take several minutes)

```
$ vagrant box add vStone/centos-7.x-puppet.3.x
==> box: Loading metadata for box 'vStone/centos-7.x-puppet.3.x'
    box: URL: https://atlas.hashicorp.com/vStone/centos-7.x-puppet.3.x
==> box: Adding box 'vStone/centos-7.x-puppet.3.x' (v4.3.26.1) for provider: virtualbox
    box: Downloading: https://atlas.hashicorp.com/vStone/boxes/centos-7.x-puppet.3.x/versions/4.3.26.1/providers/virtualbox.box
==> box: Successfully added box 'vStone/centos-7.x-puppet.3.x' (v4.3.26.1) for 'virtualbox'!
```

Download a `Vagrantfile`

```
$ curl https://raw.githubusercontent.com/ericcf/guides/master/Vagrantfile > Vagrantfile
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3220  100  3220    0     0   1117      0  0:00:02  0:00:02 --:--:--  1117
```

Boot the VM image. Note: you will receive errors about a tty requirement. It's expected!

```
$ vagrant up
...
==> default: sudo: sorry, you must have a tty to run sudo
The SSH command responded with a non-zero exit status. Vagrant
assumes that this means the command failed. The output for this command
should be in the log above. Please read the output to determine what
went wrong.
```

Configure SElinux permissive mode

```
$ vagrant ssh -c "sudo sed -i -e 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config"
Connection to 127.0.0.1 closed.
```

SSH into the guest OS

```
$ vagrant ssh
Last login: Wed May  6 19:32:18 2015 from 10.0.2.2
[vagrant@centos-7 ~]$
```

Remove TTY requirement by editing `/etc/sudoers` with the command `sudo visudo` and commenting the following lines:

```
#Defaults    requiretty
#Defaults   !visiblepw
```

Fix folder mounting of Guest Additions

```
[vagrant@centos-7 ~]$ sudo ln -s /opt/VBoxGuestAdditions-4.3.24/lib/VBoxGuestAdditions /usr/lib/VBoxGuestAdditions
```

Restart the vm and provision. This installs required packages and dependencies, and will take a while.

```
[vagrant@centos-7 ~]$ exit
logout
Connection to 127.0.0.1 closed.
$ vagrant reload --provision
...
==> default: Complete!
```

Install a Vagrant plugin to keep Guest Additions up to date

```
$ vagrant plugin install vagrant-vbguest
Installing the 'vagrant-vbguest' plugin. This can take a few minutes...
Installed the plugin 'vagrant-vbguest (0.10.0)'!
$ vagrant reload
```

## Install PostgreSQL

edit the CentOS-Base.repo file

```
$ vagrant ssh
[vagrant@centos-7 ~]$ sudo vi /etc/yum.repos.d/CentOS-Base.repo
```

add `exclude=postgresql*` to the `[base]` and `[updates]` sections

Install Postgres dependencies

```
[vagrant@centos-7 ~]$ sudo yum -y install http://yum.postgresql.org/9.4/redhat/rhel-6-x86_64/pgdg-redhat94-9.4-1.noarch.rpm
...
Installed:
  pgdg-redhat94.noarch 0:9.4-1

Complete!
[vagrant@centos-7 ~]$ sudo yum -y install postgresql94-server postgresql94-contrib
...
Dependency Installed:
  libxslt.x86_64 0:1.1.28-5.el7                           postgresql94.x86_64 0:9.4.1-1PGDG.rhel7                           postgresql94-libs.x86_64 0:9.4.1-1PGDG.rhel7

Complete!
```

Configure Postges to start when the server boots

```
[vagrant@centos-7 ~]$ sudo systemctl enable postgresql-9.4
ln -s '/usr/lib/systemd/system/postgresql-9.4.service' '/etc/systemd/system/multi-user.target.wants/postgresql-9.4.service'
```

Start Postgres server

```
[vagrant@centos-7 ~]$ sudo /usr/pgsql-9.4/bin/postgresql94-setup initdb
Initializing database ... OK

[vagrant@centos-7 ~]$ sudo systemctl start postgresql-9.4
```

Add `vagrant` role to Postgres as superuser with no password

```
[vagrant@centos-7 ~]$ sudo su - postgres
-bash-4.2$ createuser -d -r -s vagrant
-bash-4.2$ exit
logout
[vagrant@centos-7 ~]$
```

You should be able to log in without a password

```
[vagrant@centos-7 ~]$ psql -d postgres
psql (9.4.1)
Type "help" for help.

postgres=#
```

Type `\d` + `enter/return` or `control` + `d` to exit `psql`.

## Install RVM

Install public key

```
[vagrant@centos-7 ~]$ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
...
gpg:               imported: 1  (RSA: 1)
```

Install RVM stable with Ruby (this will likely take several minutes)

```
[vagrant@centos-7 ~]$ \curl -sSL https://get.rvm.io | bash -s stable --ruby
...
  * To start using RVM you need to run `source /home/vagrant/.rvm/scripts/rvm`
    in all your open shell windows, in rare cases you need to reopen all shell windows.
```

## Install NodeJS

```
[vagrant@centos-7 ~]$ wget http://nodejs.org/dist/v0.12.0/node-v0.12.0.tar.gz
...
2015-05-07 12:48:29 (789 KB/s) - ‘node-v0.12.0.tar.gz’ saved [19096897/19096897]

[vagrant@centos-7 ~]$ tar xzvf node-v* && cd node-v*
...
node-v0.12.0/.gitattributes
[vagrant@centos-7 node-v0.12.0]$ ./configure; make
...
ln -fs out/Release/node node
[vagrant@centos-7 node-v0.12.0]$ sudo make install
...
installing /usr/local/include/node/zlib.h
[vagrant@centos-7 node-v0.12.0]$ cd
[vagrant@centos-7 ~]$
```

## Install Phusion Passenger

```
[vagrant@centos-7 ~]$ gem install passenger
...
2 gems installed
[vagrant@centos-7 ~]$ sudo chmod o+x "/home/vagrant"
```

Temporarily increase available memory:

```
[vagrant@centos-7 ~]$ sudo dd if=/dev/zero of=/swap bs=1M count=1024
...
1073741824 bytes (1.1 GB) copied, 1.97347 s, 544 MB/s
[vagrant@centos-7 ~]$ sudo mkswap /swap
Setting up swapspace version 1, size = 1048572 KiB
no label, UUID=104e8f43-3891-44d5-9057-9f5d6c79041f
[vagrant@centos-7 ~]$ sudo swapon /swap
swapon: /swap: insecure permissions 0644, 0600 suggested.

```

Run Passenger installation, and follow the instructions (this will take several minutes)

```
[vagrant@centos-7 ~]$ passenger-install-apache2-module
```

add configuration file as instructed: `/etc/httpd/conf.d/passenger.conf`

## Configure and start Apache

```
[vagrant@centos-7 ~]$ sudo systemctl enable httpd
[vagrant@centos-7 ~]$ sudo systemctl start httpd
````

## Clean up

```
[vagrant@centos-7 ~]$ rm -rf node-*
```

## Install Rails

```
[vagrant@centos-7 ~]$ echo "gem: --no-document" > ~/.gemrc
[vagrant@centos-7 ~]$ gem install rails
```

## Set up port forwarding

[Follow these instructions](https://gist.github.com/altryne/60693da48d1b776c8265)
