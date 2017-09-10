# upstream-testing-vagrant
Starts a VM to run [upstream tests](https://upstreamfirst.fedorainfracloud.org/browse/projects/) on Fedora using Vagrant.

## Install virtualbox or libvirt provider according to your preference

The Vagrantfile supports virtualbox and libvirt provider. Use whichever you prefer.

## Install Vagrant
Install required packages run Vagrant on Fedora

    sudo dnf install -y libvirt vagrant vagrant-libvirt

Download the [Vangrantfile](https://github.com/bgoncalv/upstream-testing-vagrant/raw/master/Vagrantfile) to a directory where vagrant will be executed from.
> All vagrant commands should be executed from this directory

Allow libvirt to run without asking for password

    sudo gpasswd -a ${USER} libvirt
    newgrp libvirt

## Install sshfs plugin for sharing tests and scripts directories

Run this command to install the ssfs plugin to Vagrant.

    sudo yum -y install fuse-sshfs
    vagrant plugin install sshfs

## /tests and /scripts directories

The tests directory is synchronized inside VM to /mnt/tests. It is expected to use this folder for the ported beakerlib tests. This way you can directly edit the tests code on you local machine.

The scripts directory is synchronized inside VM to /scripts and it currently contains these scripts:

    migrate-fedora-tests - run this script in a test directory to apply safe fixes to beakerlib tests for Fedora and also print warnings about issues that cannot be automatically fixed

## Starting VM
Run vagrant command to start a VM, the first time might take a while to download the image.
Execute vagrant command within same directory where Vagrantfile is located.

    vagrant up

## Automatically run tests for a package
It is possible to automatically start the VM and automatically run the test for a package.

    vagrant --test-name=<package-name> up

Eg.

    vagrant --test-name=diffutils up


## Connecting to the VM
Connect via ssh to the VM, all tools required to run upstream tests will be already installed.

    vagrant ssh


## Removing the VM
Once the VM is not needed it should be removed.

    vagrant destroy
