# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'getoptlong'

opts = GetoptLong.new(
  [ '--test-name', GetoptLong::OPTIONAL_ARGUMENT ]
)

test_name=''

opts.each do |opt, arg|
  case opt
    when '--test-name'
      test_name=arg
  end
end

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  #config.vm.box = "base"
  #config.vm.box = "fedora/25-cloud-base"
  config.vm.box = "fedora/26"
  config.vm.box_url = "https://download.fedoraproject.org/pub/fedora/linux/releases"\
                      "/26/CloudImages/x86_64/images/Fedora-Cloud-Base-Vagrant-26-1"\
                      ".5.x86_64.vagrant-libvirt.box"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  config.vm.provider "libvirt" do |libvirt|
        libvirt.memory = 1024
        libvirt.cpu_mode = "host-model"
  end
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
  config.vm.provision "shell", inline: <<-SHELL
        set -x
        dnf install -y git

        #install ansible test runner prerequisites
        dnf install -y ansible python2-dnf libselinux-python

        dnf copr enable -y merlinm/standard-test-roles
        dnf --enablerepo=updates-testing install -y  standard-test-roles

        #If test name was given run the tests for it
        if [ -n "#{test_name}" ]; then
            git clone https://upstreamfirst.fedorainfracloud.org/#{test_name}.git
            cd #{test_name}
            sudo ansible-playbook test_local.yml -e artifacts=$PWD/artifacts
            cat $PWD/artifacts/test.log
            #Check if any test did not pass
            grep -ve ^PASS $PWD/artifacts/test.log
            if [ $? -eq 1 ]; then
                echo "PASS: all tests passed."
                exit 0
            fi
            echo "FAIL: At least one test did not PASS."
            exit 1
        fi
    SHELL
end
