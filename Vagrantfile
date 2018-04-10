# -*- mode: ruby -*-
# vi: set ft=ruby :
# Based off instructions found here:
# https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FslSge
# Additional Requirements:
# https://github.com/devopsgroup-io/vagrant-hostmanager

# Configurable options!
# The URL containing the RPMs.
baseurl = "https://arc.liv.ac.uk/downloads/SGE/releases/"
# The version number.
version = "8.1.6"
# The release number.
release = "1"

Vagrant.configure("2") do |config|
  config.vm.box = "centos/6"

  # Set number of cores for master and nodes
  mastercores = 2
  nodecores = 1

  # Get host memory.
  # From: https://stefanwrobel.com/how-to-make-vagrant-performance-not-suck
  host = RbConfig::CONFIG['host_os']

  mem = 1024
  if host =~ /darwin/
    # sysctl returns Bytes and we need to convert to MB
    mem = `sysctl -n hw.memsize`.to_i / 1024
  elsif host =~ /linux/
    # meminfo shows KB and we need to convert to MB
    mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i 
  elsif host =~ /mswin|mingw|cygwin/
    # Windows code via https://github.com/rdsubhas/vagrant-faster
    mem = `wmic computersystem Get TotalPhysicalMemory`.split[1].to_i / 1024
  end

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = false
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  # Define the master node.
  config.vm.define "master" do |master|
    # Set memory and CPU allocation.
    master.vm.provider "virtualbox" do |v|
      v.memory = mem / 1024/ 4
      v.cpus = mastercores
    end
    # Shared directory for data processing/storage.
    master.vm.synced_folder ".", "/data", type: "nfs"
    # Add IP Address for private LAN.
    master.vm.network "private_network", ip: "192.168.2.2"
    # Define hostname.
    master.vm.hostname = "master"
  end
  
  # Define several secondary nodes.
  config.vm.define "nodeA" do |nodeA|
    nodeA.vm.provider "virtualbox" do |v|
      v.memory = mem / 1024/ 8
      v.cpus = nodecores
    end
    nodeA.vm.synced_folder ".", "/data", type: "nfs"
    nodeA.vm.network "private_network", ip: "192.168.2.3"
    nodeA.vm.hostname = "nodeA"
  end
  config.vm.define "nodeB" do |nodeB|
    nodeB.vm.provider "virtualbox" do |v|
      v.memory = mem / 1024/ 8
      v.cpus = nodecores
    end
    nodeB.vm.synced_folder ".", "/data", type: "nfs"
    nodeB.vm.network "private_network", ip: "192.168.2.4"
    nodeB.vm.hostname = "nodeB"
  end

  # Provision master and secondary nodes.
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "site.yml"
    ansible.groups = {
      "masters" => ["master"],
      "nodes" => ["nodeA","nodeB"],
    }
  end
end
