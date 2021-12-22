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
  config.vm.box = "centos/7"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  config.vm.network "forwarded_port", guest: GUEST_PORT, host: HOST_PORT, host_ip: "127.0.0.1"

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
    echo '#### Open the aplication at:  http://127.0.0.1:#{HOST_PORT}/index.html ####'
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
