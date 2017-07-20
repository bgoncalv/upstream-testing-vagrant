# upstream-testing-vagrant
Starts a VM to run [upstream tests](https://upstreamfirst.fedorainfracloud.org/browse/projects/) on Fedora using Vagrant.


## Install Vagrant
Install required packages run Vagrant on Fedora

    sudo dnf install -y libvirt vagrant vagrant-libvirt

Download the [Vangrantfile](https://github.com/bgoncalv/upstream-testing-vagrant/blob/master/Vagrantfile) to a directory where vagrant will be executed from.
> All vagrant commands should be executed from this directory

Allow libvirt to run without asking for password

    sudo gpasswd -a ${USER} libvirt
    newgrp libvirt

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
