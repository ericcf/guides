# How to set up a Rails app development environment on OSX using Vagrant

## Install VirtualBox

Download it here: http://download.virtualbox.org/virtualbox/4.3.24/VirtualBox-4.3.24-98716-OSX.dmg

Install VirtualBox and start the application

Download the extension pack here: http://dlc-cdn.sun.com/virtualbox/4.3.24/Oracle_VM_VirtualBox_Extension_Pack-4.3.24.vbox-extpack

Double click the extension pack to install it

## Install Vagrant

Download it here: https://dl.bintray.com/mitchellh/vagrant/vagrant_1.7.2.dmg

## Set up CentOS

Change to the directory where you'll store the configuration for your VM

Download a CentOS 6.5 Vagrant box

```
$ vagrant box add vStone/centos-7.x-puppet.3.x
```

Download a `Vagrantfile`

```
$ curl https://raw.githubusercontent.com/ericcf/guides/master/Vagrantfile > Vagrantfile
```

Boot the VM image. Note: you will receive errors about a tty requirement. It's expected!

```
$ vagrant up
```

Configure SElinux permissive mode

```
$ vagrant ssh -c "sudo sed -i -e 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config"
Connection to 127.0.0.1 closed.
```

SSH into the guest OS

```
$ vagrant ssh
```

Remove TTY requirement by editing `/etc/sudoers` with the command `sudo visudo` and commenting the following lines:

```
#Defaults    requiretty
#Defaults   !visiblepw
```

Fix folder mounting of Guest Additions

```
[vagrant@centos-7 ~]$ sudo ln -s /opt/VBoxGuestAdditions-4.3.18/lib/VBoxGuestAdditions /usr/lib/VBoxGuestAdditions
```

Restart the vm and provision. This installs required packages and dependencies, and will take a while.

```
[vagrant@centos-7 ~]$ exit
logout
Connection to 127.0.0.1 closed.
$ vagrant reload --provision
```

Install a Vagrant plugin to keep Guest Additions up to date

```
$ vagrant plugin install vagrant-vbguest
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
[vagrant@centos-7 ~]$ sudo yum -y install postgresql94-server postgresql94-contrib
```

Configure Postges to start when the server boots

```
[vagrant@centos-7 ~]$ sudo systemctl enable postgresql-9.4
```

Start Postgres server

```
[vagrant@centos-7 ~]$ sudo /usr/pgsql-9.4/bin/postgresql94-setup initdb
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

## Install RVM

Install public key

```
[vagrant@centos-7 ~]$ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
```

Install RVM stable with Ruby

```
\curl -sSL https://get.rvm.io | bash -s stable --ruby
```

## Install NodeJS

`wget http://nodejs.org/dist/v0.12.0/node-v0.12.0.tar.gz`

`tar xzvf node-v* && cd node-v*`

`./configure; make`

`sudo make install`

## Install Phusion Passenger

```
gem install passenger
sudo chmod o+x "/home/vagrant"
```

Temporarily increase available memory:

```
sudo dd if=/dev/zero of=/swap bs=1M count=1024
sudo mkswap /swap
sudo swapon /swap
```

```
passenger-install-apache2-module
```

add configuration file as instructed: `/etc/httpd/conf.d/passenger.conf`

## Configure and start Apache

```
sudo systemctl enable httpd
sudo systemctl start httpd
````
