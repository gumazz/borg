# -*- mode: ruby -*-
# vi: set ft=ruby :
# export VAGRANT_EXPERIMENTAL="disks"
disk = './secondDisk.vdi'

Vagrant.configure("2") do |config|

config.vm.define "server" do |server|
  server.vm.box = 'generic/ubuntu2204'
  server.vm.host_name = 'server'
  server.vm.network :private_network, ip: "192.168.11.160" 
  server.vm.disk :disk, size: "2GB", name: "backup_storage"
  server.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.name = 'server'
    unless File.exist?(disk)
      vb.customize ['createhd', '--filename', disk, '--variant', 'Fixed', '--size', 2048]
    end
      vb.customize ['storageattach', 'server',  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', disk]
  end
  server.vm.provision "ansible" do |ansible|
    ansible.playbook = "provision_server.yml"
    ansible.inventory_path = "hosts"
    ansible.host_key_checking = "false"
    ansible.limit = "all"
  end
  server.vm.provision "shell", inline: <<-SHELL
	mkdir /var/backup
  	mkfs -t ext4 /dev/sdb
	echo "/dev/sdb   /var/backup   ext4   defaults   0   0" >> /etc/fstab
	mount /dev/sdb /var/backup
	rm -rf /var/backup/*
	chown borg:borg /var/backup
	chmod 755 /var/backup
  SHELL
end

  config.vm.define "client" do |client|
    client.vm.box = 'generic/ubuntu2204'
    client.vm.host_name = 'client'
    client.vm.network :private_network, ip: "192.168.11.150"
    client.vm.provider :virtualbox do |vb|
      vb.memory = "2048"
    end
  client.vm.provision "ansible" do |ansible|
    ansible.playbook = "provision_client.yml"
    ansible.inventory_path = "hosts"
    ansible.host_key_checking = "false"
    ansible.limit = "all"
  end
  end
end
