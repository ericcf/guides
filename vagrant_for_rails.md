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
