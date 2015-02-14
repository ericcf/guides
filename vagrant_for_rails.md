# How to set up a Rails app development environment on OSX using Vagrant

## Install VirtualBox

Download it here: http://download.virtualbox.org/virtualbox/4.3.22/VirtualBox-4.3.22-98236-OSX.dmg

Install VirtualBox and start the application

Download the extension pack here: http://download.virtualbox.org/virtualbox/4.3.22/Oracle_VM_VirtualBox_Extension_Pack-4.3.22-98236.vbox-extpack

Double click the extension pack to install it

## Install Vagrant

Download it here: https://dl.bintray.com/mitchellh/vagrant/vagrant_1.7.2.dmg

## Set up CentOS

Change to the directory where you'll store the configuration for your VM

Download a CentOS 6.5 Vagrant box with the command:
`vagrant box add vStone/centos-7.x-puppet.3.x`

Generate a `Vagrantfile` with the command: `vagrant init vStone/centos-7.x-puppet.3.x`

Configure a forwarded port on the guest machine by editing line 25 of your Vagrantfile to look like:
```
config.vm.network "forwarded_port", guest: 3000, host: 3000
```

Boot the VM image with the command: `vagrant up`

ssh into the guest OS: `vagrant ssh`

## Install PostgreSQL

edit the CentOS-Base.repo file: `sudo vi /etc/yum.repos.d/CentOS-Base.repo`

add `exclude=postgresql*` to the `[base]` and `[updates]` sections

Download Postgres rpms: `sudo rpm -Uvh http://yum.postgresql.org/9.4/redhat/rhel-7-x86_64/pgdg-centos94-9.4-1.noarch.rpm`

Install Postgres packages: `sudo yum -y install postgresql94 postgresql94-devel postgresql94-server postgresql94-libs postgresql94-contrib`

Configure Postges to start when the server boots: `sudo systemctl enable postgresql-9.4`

Start Postgres:
```
sudo /usr/pgsql-9.4/bin/postgresql94-setup initdb
sudo systemctl start postgresql-9.4
```

Add `vagrant` role to Postgres as superuser with no password:
```
sudo su - postgres
createuser -d -r -s vagrant
exit
```

You should be able to log in without a password: `psql -d postgres`

## Install RVM

Install public key: `gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3`

Install RVM stable with Ruby: `\curl -sSL https://get.rvm.io | bash -s stable --ruby`

## Install NodeJS

`wget http://nodejs.org/dist/v0.12.0/node-v0.12.0.tar.gz`

## Install Phusion Passenger

Install public key: `rpm --import http://passenger.stealthymonkeys.com/RPM-GPG-KEY-stealthymonkeys.asc`

