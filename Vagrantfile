# -*- mode: ruby -*-
# vi: set et ts=2 sw=2 ft=ruby :

def attach_disk(config, prefix, disk_num, size)
  filename = "#{prefix}#{disk_num}.vdi"
  config.vm.provider "virtualbox" do |vb|
    if !File.exist?(filename) 
      vb.customize ['createhd', '--filename', filename, '--size', (size * 1024).floor, '--variant', 'fixed']
      vb.customize ['modifyhd', filename, '--type', 'shareable']
    end

    vb.customize ['storageattach', :id, '--storagectl', 'SATAController', '--port', disk_num + 2, '--device', 0, '--type', 'hdd', '--medium', filename]
  end
end

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 2
    v.customize ["storagectl", :id, "--name", "SATAController", "--add", "sata"]
  end

  mdt_count = 1
  mdt_size = 0.25
  config.vm.define "mds0", primary: true do |h|
    h.vm.network :private_network, ip: "10.3.2.1"

    (1..mdt_count).each do |disk_num|
      attach_disk(config, "mdt", disk_num, mdt_size)
    end
  end

  ost_count = 2
  ost_size = 0.25
  config.vm.define "oss0", primary: true do |h|
    h.vm.network :private_network, ip: "10.3.2.2"

    (1..ost_count).each do |disk_num|
      attach_disk(config, "ost", disk_num, ost_size)
    end

  end

  config.vm.define "c0", primary: true do |h|
    h.vm.network :private_network, ip: "10.3.2.3"
  end

  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.groups = {
      "mds" => ["mds0"],
      "oss" => ["oss0"],
      "client" => ["c0"]
    }
  end
end
