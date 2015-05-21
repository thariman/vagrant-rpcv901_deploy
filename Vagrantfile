# -*- mode: ruby -*-

# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

$script = <<SCRIPT
cat << \EOF >> /etc/hosts
172.29.236.7 deploy1
172.29.236.2 021579-infra01
172.29.236.5 021579-logging01
172.29.236.10 021579-compute001
EOF
SCRIPT

$start = <<SCRIPT
cat << EOF >> /root/start1.sh
#!/usr/bin/env bash

set -o errexit
set -o xtrace

cd /opt/os-ansible-deployment/rpc_deployment
fping 172.29.236.2 172.29.236.5 172.29.236.10
ansible-playbook -v -e @/etc/rpc_deploy/user_variables.yml playbooks/setup/host-setup.yml
ansible-playbook -v -e @/etc/rpc_deploy/user_variables.yml playbooks/infrastructure/haproxy-install.yml
ansible-playbook -v -e @/etc/rpc_deploy/user_variables.yml playbooks/openstack/openstack-setup.yml
EOF
SCRIPT

$proxy = <<SCRIPT
echo 'Acquire::http { Proxy "http://10.64.200.100:8000"; };' > /etc/apt/apt.conf.d/10mirror
SCRIPT

$aptchange = <<SCRIPT
sed -i "s#//kambing.ui.ac.id#//buaya.klas.or.id#g" /etc/apt/sources.list
apt-get -qq update
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = "trusty64"
  config.cache.enable :apt

  # Turn off shared folders
  config.vm.synced_folder ".", "/vagrant", id: "vagrant-root", disabled: true

  # Begin deploy1
  config.vm.define "deploy1" do |deploy1_config|
    deploy1_config.vm.hostname = "deploy1"

    deploy1_config.vm.provision "shell", inline: $script
    deploy1_config.vm.provision "shell", inline: $proxy
    deploy1_config.vm.provision "shell", inline: $aptchange
    deploy1_config.vm.provision "shell", inline: $start

    # eth1
    deploy1_config.vm.network "private_network", ip: "172.29.236.7"
    # eth2
    deploy1_config.vm.network "private_network", ip: "172.29.240.7"
    
    deploy1_config.vm.provider "vmware_fusion" do |v|
        v.vmx["memsize"] = "2048"
        v.vmx["numvcpus"] = "1"
    end

    deploy1_config.vm.provider :libvirt do |v|
        v.memory = 2048
        v.cpus = 1
    end

    deploy1_config.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--memory", "2048"]
        v.customize ["modifyvm", :id, "--cpus", "1"]
        v.customize ["modifyvm", :id, "--nicpromisc4", "allow-all"]
    end
    
    deploy1_config.vm.provision "ansible" do |ansible|
    	ansible.extra_vars = { ansible_ssh_user: 'vagrant' }
        ansible.playbook = "provisioning/base.yml"
    end
  end
  # End deploy1
end
