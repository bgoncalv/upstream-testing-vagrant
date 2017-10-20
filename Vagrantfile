# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'getoptlong'

opts = GetoptLong.new(
  [ '--test-name', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--command', '-c', GetoptLong::OPTIONAL_ARGUMENT ]
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

  config.ssh.username = 'root'
  config.ssh.insert_key = false

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  #config.vm.box = "base"
  #config.vm.box = "fedora/25-cloud-base"
  #config.vm.box = "fedora/26"
  #config.vm.box_url = "https://download.fedoraproject.org/pub/fedora/linux/releases"\
  #                    "/26/CloudImages/x86_64/images/Fedora-Cloud-Base-Vagrant-26-1"\
  #                    ".5.x86_64.vagrant-libvirt.box"
  config.vm.box = "fedora/rawhide"
  config.vm.box_url = "https://download.fedoraproject.org/pub/fedora/linux/development"\
                      "/rawhide/CloudImages/x86_64/images/"\
                      "Fedora-Cloud-Base-Vagrant-Rawhide-20171016.n.0.x86_64.vagrant-libvirt.box"

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
  config.vm.synced_folder ".", "/vagrant", disabled: true

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
        libvirt.memory = 6144
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
#        echo "vagrant" | passwd --stdin root
        dnf install --best -y git

        #install ansible test runner prerequisites
        dnf install --best -y ansible python2-dnf libselinux-python

        dnf copr enable -y merlinm/standard-test-roles
        dnf --enablerepo=updates-testing install --best -y  standard-test-roles
        if [ $? -ne 0 ]; then
            dnf install --best -y standard-test-roles
            if [ $? -ne 0 ]; then
                echo "FAIL: Could not install standard-test-roles"
                exit 1;
            fi
        fi

        #WORKAROUND PR#73
        dnf install -y patch
        cd /etc/ansible/
        curl -sS https://pagure.io/standard-test-roles/pull-request/73.patch > 73.patch
        patch -p1 < 73.patch
        cd -

        #If test name was given run the tests for it
        if [ -n "#{test_name}" ]; then
            git clone https://upstreamfirst.fedorainfracloud.org/#{test_name}.git
            if [ $? -ne 0 ]; then
                echo "FAIL: Could not clone repo for #{test_name}"
                exit 1
            fi
            cd #{test_name}

            #Check if tests have any reference to bugzillas
            #using \\< and \\> to match whole words only
            bz_regex="\\(\\<bug\\>\\|bz\\)\\(_\\|#\\)*[[:digit:]]\\{4,9\\}"
            #test case names should not have reference to bugzilla
            find . -not -iwholename '*.git*' | grep -i -e "$bz_regex"
            if [ $? -eq 0 ]; then
                echo "FAIL: It looks like there is bugzilla reference on test case name"
                exit 1
            fi
            bz_regex="\\(\\<bug\\>\\|bz\\|bugzilla.redhat.com/show_bug.cgi?id=\\)\\([[:space:]]\\|#\\|:\\)*[[:digit:]]\\{4,9\\}"
            #only interested on BZ numbers
            bz_nr=$(grep --exclude-dir=.git -r -i -o -e "$bz_regex" . | grep -o -i -e "[[:digit:]]\\{4,9\\}" | sort | uniq)
            if [ ! -z "$bz_nr" ]; then
                #Check if there is reference to private bugs
                for bz in $bz_nr; do
                    echo "Checking if BZ($bz) is private"
                    curl -s https://bugzilla.redhat.com/show_bug.cgi?id=$bz | grep "<title>Access Denied</title>"
                    if [ $? -eq 0 ]; then
                        echo "FAIL: bugzilla ($bz) is private!"
                        exit 1
                    fi
                done
            fi

            cve_regex="\\(CVE\\)\\([[:space:]]\\|#\\|:\\|-\\)*[[:digit:]]\\{4\\}"
            grep --exclude-dir=.git -r -i -e "$cve_regex" .
            if [ $? -ne 1 ]; then
                echo "FAIL: It seems there are references to CVE!"
                exit 1
            fi


            ANSIBLE_INVENTORY=$(test -e inventory && echo inventory || echo /usr/share/ansible/inventory)
            TEST_SUBJECTS="" 
            TEST_ARTIFACTS=$PWD/artifacts
            ansible-playbook --tags classic tests.yml
            if [ $? -ne 0 ]; then
                echo "FAIL: Could not execute ansible-playbook"
                exit 1
            fi
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
