# -*- mode: ruby -*-
# vi: set ft=ruby :

# Parameters for easy reconfiguration
# Port to use to access the server on the external host
HOST_PORT=8086

# Constants - these should not need to change
# Internal port for server
GUEST_PORT=8080
# The directory for the server in the guest
SERVER_DIR="/home/vagrant/server/"
# The log file for the server, since it needs to run in the background
SERVER_LOG_FILE="/home/vagrant/server.log"
# The host directory for the compiled Royale project
COMPILED_DIR="bin/js-debug"


# Check if the project is compiled
if !File.exist?("#{COMPILED_DIR}/index.html") then
  abort "You must compile this project first.  In Moonshine, run Project > Build Project"
end

Vagrant.configure("2") do |config|
  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  # centos/7 had trouble with guest additions
  #config.vm.box = "centos/7"  # centos/8 is available, but requires changes to the installed packages
  # Use official minimal box from Hashicorp
  config.vm.box = "hashicorp/bionic64"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  if Vagrant::Util::Platform.windows? then
    # host_ip: "127.0.0.1" fails on Windows, but it looks like a useful security restriction in general
    # https://www.vagrantup.com/docs/networking/forwarded_ports#host_ip
    config.vm.network "forwarded_port", guest: GUEST_PORT, host: HOST_PORT
  else
    config.vm.network "forwarded_port", guest: GUEST_PORT, host: HOST_PORT, host_ip: "127.0.0.1"
  end

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "bin/js-debug",SERVER_DIR

  # Install some dependencies
  config.vm.provision "shell", privileged: true, inline: <<-SHELL
    apt-get -y install python
    
    # Debugging tools
    apt-get install -y telnet
  SHELL
  
  # Define a basic service for the Python server
  # https://linuxconfig.org/how-to-write-a-simple-systemd-service
  config.vm.provision "shell", privileged: true, inline: <<-SHELL
# write the service
echo "[Unit]
Description=Start a minimal Python server to run the demo application.

[Service]
Type=simple
ExecStart=/usr/bin/python -m SimpleHTTPServer #{GUEST_PORT}
WorkingDirectory=#{SERVER_DIR}
StandardOutput=file:/home/vagrant/output.log
StandardError=file:/home/vagrant/error.log

[Install]
WantedBy=multi-user.target
" > /etc/systemd/system/demo_server.service

# setup permissions
chmod 755 /etc/systemd/system/demo_server.service
chown root:root /etc/systemd/system/demo_server.service
  SHELL
  

  # Kill the server if it is already running. This is useful for reloading the server with `vagrant up` or `vagrant reload`
  config.vm.provision "shell",privileged:true,run:"always",inline: "pkill python || true"
  
  # Start as service
  config.vm.provision "shell",privileged:false,run:"always",inline: "sudo systemctl start demo_server.service"

  # Notify the user how to access the server
  config.vm.provision "shell",privileged:false,run:"always",inline: <<-SHELL
    echo '#### Server is starting. See log at #{SERVER_LOG_FILE} ####'
    echo '#### Open the application at:  http://127.0.0.1:#{HOST_PORT}/index.html ####'
  SHELL


end
