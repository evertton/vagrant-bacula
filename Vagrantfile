# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "generic/debian9"
  config.vm.box_check_update = true
  #config.vm.network "forwarded_port", guest: 80, host: 8080
  #config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.provision "shell", inline: <<-SHELL
    set -x
    apt update -y && apt upgrade -y
    apt install -y nano 
  SHELL

  config.vm.define "nfs", primary: true do |s|
      s.vm.hostname = "bacula-nfs"
      s.vm.netwok "private_network", ip: "192.168.111.9"
      s.vm.provision "shell", inline: <<-SHELL
        
      SHELL
  end
  
  config.vm.define "sd", primary: true do |s|
      s.vm.hostname = "bacula-sd"
      s.vm.netwok "private_network", ip: "192.168.111.10"
      s.vm.provision "shell", inline: <<-SHELL
        
      SHELL
  end
  
  config.vm.define "dir", primary: true do |s|
      s.vm.hostname = "bacula-dir"
      s.vm.netwok "private_network", ip: "192.168.111.11"
      s.vm.provision "shell", inline: <<-SHELL
        
      SHELL
  end
  
  config.vm.define "fd1", primary: true do |s|
      s.vm.hostname = "bacula-fd1"
      s.vm.netwok "private_network", ip: "192.168.111.12"
      s.vm.provision "shell", inline: <<-SHELL
        
      SHELL
  end
  
  config.vm.define "fd2", primary: true do |s|
      s.vm.hostname = "bacula-fd2"
      s.vm.netwok "private_network", ip: "192.168.111.13"
      s.vm.provision "shell", inline: <<-SHELL
        
      SHELL
  end
end
