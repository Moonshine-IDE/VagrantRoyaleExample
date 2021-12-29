# -*- mode: ruby -*-
# vi: set ft=ruby :

# Parameters for easy reconfiguration
# Internal port for server
GUEST_PORT=8080
# Port to use to access the server on the external host
HOST_PORT=8080
# The directory for the server in the guest
SERVER_DIR="/home/vagrant/server/"
# The log file for the server, since it needs to run in the background
SERVER_LOG_FILE="/home/vagrant/server.log"

Vagrant.configure("2") do |config|
  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "centos/7"  # centos/8 is available, but requires changes to the installed packages

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  config.vm.network "forwarded_port", guest: GUEST_PORT, host: HOST_PORT, host_ip: "127.0.0.1"

  # This is the solution for VirtualBox Guest Additions used here:  https://github.com/DominoIDP/domino_idp
  # However, I found that this gave a "mount: unknown filesystem type 'vboxsf'" error on the first execution, and then worked after trying "vagrant up" instead.   
#  # OS upgrade - needed for vbguest
#  config.vm.provision "shell", name: "Upgrade Linux so VirtualBox Guest Additions will install", privileged:true, inline: "yum -y upgrade" 
#  
#  #reboot after upgrade
#  #https://superuser.com/questions/1338429/how-do-i-reboot-a-vagrant-guest-from-a-provisioner
#  config.vm.provision :shell do |shell|
#    shell.privileged = true
#    shell.inline = 'echo rebooting'
#    shell.reboot = true
#  end  
#  
#  ## WORKAROUND for VirtualBox Guest Additions not installing properly - have not tried all of these options yet
#  #read this and try to get the Vagrantfile itself to do this
#  #https://github.com/hashicorp/vagrant/issues/8374
#  #From https://gist.github.com/adaroobi/fea2727be6ae3d9c446767f813146f93
#  #
#  #https://www.serverlab.ca/tutorials/virtualization/how-to-auto-upgrade-virtualbox-guest-additions-with-vagrant/
#  #might be able to get this alternate approach to work in the Vagrantfile:
#  #vagrant plugin install vagrant-vbguest
#  #vagrant vbguest --do install --no-cleanup
#  if Vagrant.has_plugin?("vagrant-vbguest") then
#    config.vbguest.auto_update = false
#  else
#    puts "Missing vbguest plugin.  Install using 'vagrant plugin install vagrant-vbguest'"
#  end
#  config.vm.provision "shell", name: "WORKAROUND for VirtualBox Guest Additions.", privileged: true, inline: <<-SHELL
#    VBOX_VERSION_ON_HOST_OS=6.1.30     #This should be set to YOUR host OS release of VirtualBox - use "VBoxManage -v" and trim the "r14..." suffix.
#	
#    rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
#    #rpm -Uvh http://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/d/dkms-2.6.1-1.el7.noarch.rpm
#    rpm -Uvh https://download-ib01.fedoraproject.org/pub/epel/7/aarch64/Packages/d/dkms-2.7.1-1.el7.noarch.rpm
#	yum -y install wget perl gcc dkms kernel-devel kernel-headers make bzip2
#    wget http://download.virtualbox.org/virtualbox/${VBOX_VERSION_ON_HOST_OS}/VBoxGuestAdditions_${VBOX_VERSION_ON_HOST_OS}.iso
#	
#    mkdir /media/VBoxGuestAdditions
#    mount -o loop,ro VBoxGuestAdditions_${VBOX_VERSION_ON_HOST_OS}.iso /media/VBoxGuestAdditions
#    sh /media/VBoxGuestAdditions/VBoxLinuxAdditions.run
#	#ugh.. this is still no matching kernel version!
#	#https://www.binarytides.com/check-virtualbox-guest-additions-installed-linux-guest/
#	#lsmod | grep -i vbox
#	#https://www.dev2qa.com/how-to-resolve-virtualbox-guest-additions-kernel-headers-not-found-for-target-kernel-error/
#	#https://unix.stackexchange.com/questions/272638/why-cant-i-find-kernel-headers-on-centos-7-when-trying-to-install-virtualbox-gu/505021
#	#https://bugs.centos.org/view.php?id=17845
#    #yum install “kernel-devel-uname-r == $(uname -r)
#	#uname -r; ls /usr/src/kernels/
#	
#    #rm -f VBoxGuestAdditions_${VBOX_VERSION_ON_HOST_OS}.iso
#    #umount /media/VBoxGuestAdditions
#    #mdir /media/VBoxGuestAdditions
#    unset VBOX_VERSION_ON_HOST_OS
#  SHELL


  # Install guest additions with vagrant-vbguest only
  # See more details about vagrant-vbguest here:  https://github.com/dotless-de/vagrant-vbguest/blob/main/Readme.md
  if Vagrant.has_plugin?("vagrant-vbguest") then
    # We rely on a shared folder at ~/server for this example, so we need to install the guest additions.
    # Note that it is possible to use the files in /vagrant instead, but we included this as an example of shared folder support.
    config.vbguest.auto_update = true
    # suggested for CentOS - updates the OS rather than trying to find a version for the current OS.  Reboots the instance.  
    config.vbguest.installer_options = { allow_kernel_upgrade: true } 
  else
    abort "Missing vbguest plugin.  Install using 'vagrant plugin install vagrant-vbguest'"
  end

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "bin/js-debug",SERVER_DIR

  # Install some dependencies
  config.vm.provision "shell", privileged: true, inline: <<-SHELL
    yum -y install python
    # used to run the server in the background
    yum -y install screen
  SHELL

  # Kill the server if it is already running. This is useful for reloading the server with `vagrant up` or `vagrant reload`
  config.vm.provision "shell",privileged:true,run:"always",inline: "pkill python || true"

  # Notify the user how to access the server
  config.vm.provision "shell",privileged:false,run:"always",inline: <<-SHELL
    echo '#### Server is starting. See log at #{SERVER_LOG_FILE} ####'
    echo '#### Open the application at:  http://127.0.0.1:#{HOST_PORT}/index.html ####'
  SHELL

  # Start the server in the shared directory.
  # `run:"always"` will restart the server when rerunning `vagrant up` or `vagrant reload`
  config.vm.provision "shell",privileged:false,run:"always",inline: "cd #{SERVER_DIR}; python -m SimpleHTTPServer #{GUEST_PORT} 2>&1 | tee #{SERVER_LOG_FILE}"
  # I tried to start the server in the background instead, but the server was killed with no error.  
  #config.vm.provision "shell",privileged:false,run:"always",inline: "cd #{SERVER_DIR}; python -m SimpleHTTPServer #{GUEST_PORT} 2>&1 | tee #{SERVER_LOG_FILE} & sleep 1"
  #config.vm.provision "shell",privileged:false,run:"always",inline: "screen -dm bash -c 'cd #{SERVER_DIR}; python -m SimpleHTTPServer #{GUEST_PORT} 2>&1 | tee #{SERVER_LOG_FILE} &'"

  # Background start only:  give the server a little time to start and then print the server output
  #config.vm.provision "shell",privileged:false,run:"always",inline: "sleep 5; cat #{SERVER_LOG_FILE}"


end
