# -*- mode: ruby -*-
# vi: set ft=ruby :

# Controller config.
$controller_hostname = "controller"
$controller_ip_address = "172.31.0.1"

# Node 1 config.
$node1_hostname = "master"
$node1_ip_address = "172.31.0.2"

# Node 2 config.
$node2_hostname = "slave"
$node2_ip_address = "172.31.0.3"

# Default for machines
$vbox_cpu = 1
$vbox_memory = 512

# Sets guest environment variables.
# @see https://github.com/hashicorp/vagrant/issues/7015
$set_environment_variables = <<SCRIPT
tee "/etc/profile.d/myvars.sh" > "/dev/null" <<EOF
# Ansible environment variables.
export ANSIBLE_RETRY_FILES_ENABLED=0
EOF
SCRIPT

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  config.ssh.private_key_path = "./ansible/insecure_private_key"
  config.vm.box = "ubuntu/bionic64"
  config.vm.define $node1_hostname do |machine|
    machine.vm.provider "virtualbox" do |vbox|
      vbox.name = $node1_hostname
      vbox.memory = $vbox_memory
      vbox.cpus = $vbox_cpu
    end
    machine.vm.hostname = $node1_hostname
    machine.vm.network "private_network", ip: $node1_ip_address
  end
  config.vm.define $node2_hostname do |machine|
    machine.vm.provider "virtualbox" do |vbox|
      vbox.name = $node2_hostname
      vbox.memory = $vbox_memory
      vbox.cpus = $vbox_cpu
    end
    machine.vm.hostname = $node2_hostname
    machine.vm.network "private_network", ip: $node2_ip_address
  end
  config.vm.define $controller_hostname do |machine|
    machine.vm.provider "virtualbox" do |vbox|
      vbox.name = $controller_hostname
      vbox.memory = $vbox_memory
      vbox.cpus = $vbox_cpu
    end
    machine.vm.hostname = $controller_hostname
    machine.vm.network "private_network", ip: $controller_ip_address
    machine.vm.synced_folder "ansible", "/etc/ansible"
    machine.vm.provision "shell", inline: $set_environment_variables, \
      run: "always"
    machine.vm.provision "shell", path: "scripts/bootstrap.sh"
    machine.vm.provision "ansible_local" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.install = false
      ansible.provisioning_path = "/etc/ansible"
      ansible.playbook = ENV["ANSIBLE_PLAYBOOK"] ? ENV["ANSIBLE_PLAYBOOK"] \
        : "playbook.yml"
      ansible.inventory_path = "hosts.ini"
      ansible.become = true
      ansible.limit = ENV['ANSIBLE_LIMIT'] ? ENV['ANSIBLE_LIMIT'] : "all"
      ansible.tags = ENV['ANSIBLE_TAGS']
      ansible.verbose = ENV['ANSIBLE_VERBOSE']
    end
  end
end
