# -*- mode: ruby -*-
# vi: set ft=ruby :
# Based off instructions found here:
# https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FslSge
# Additional Requirements:
# https://github.com/devopsgroup-io/vagrant-hostmanager

$masterprov = <<SCRIPT
useradd --home /opt/sge --system sgeadmin
baseurl=http://arc.liv.ac.uk/downloads/SGE/releases/
version=8.1.6
subversion=1
for i in - -qmaster- -qmon- -execd-; do
yum install -y ${baseurl}${version}/gridengine${i}${version}-${subversion}.el6.x86_64.rpm
done
export SGE_ROOT=/opt/sge
cd $SGE_ROOT
./install_qmaster -auto /data/conf/inst_template.conf
cat <<'EOT' >> /root/.bashrc
export SGE_ROOT=/opt/sge
export SGE_CELL=default
if [ -e $SGE_ROOT/$SGE_CELL ]
then
. $SGE_ROOT/$SGE_CELL/common/settings.sh
fi
EOT
. /root/.bashrc
qconf -ah nodeA
qconf -ah nodeB
SCRIPT

$nodeprov = <<SCRIPT
useradd --home /opt/sge --system sgeadmin
baseurl=http://arc.liv.ac.uk/downloads/SGE/releases/
version=8.1.6
subversion=1
for i in - -execd-; do
yum install -y ${baseurl}${version}/gridengine${i}${version}-${subversion}.el6.x86_64.rpm
done
export SGE_ROOT=/opt/sge
cd $SGE_ROOT
./install_execd -nobincheck -auto /data/conf/inst_template.conf
SCRIPT

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
    # Provision the master node.
    master.vm.provision "shell", inline:$masterprov
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
    nodeA.vm.provision "shell", inline:$nodeprov
  end
  config.vm.define "nodeB" do |nodeB|
    nodeB.vm.provider "virtualbox" do |v|
      v.memory = mem / 1024/ 8
      v.cpus = nodecores
    end
    nodeB.vm.synced_folder ".", "/data", type: "nfs"
    nodeB.vm.network "private_network", ip: "192.168.2.4"
    nodeB.vm.hostname = "nodeB"
    nodeB.vm.provision "shell", inline:$nodeprov
  end
end
