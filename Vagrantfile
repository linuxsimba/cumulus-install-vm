# -*- mode: ruby -*-
# vi: set ft=ruby :
#
#
# Cumulus Install VM.
#
# This VagrantFile creates a VM that allows you to easily install Cumulus
# Linux and provision it using an ansible playbook.
#
# This vagrantfile only works with vagrant-libvirt. A vagrant-virtualbox
# will be available in the near future
# To convert a Box from vagrant virtualbox to vagrant-libvirt do the following
#
# sudo apt-get install software-properties-common
# sudo add-apt-repository ppa:linuxsimba/libvirt-udp-tunnel
# sudo apt-get update -y
# sudo apt-get install autoconf python-dev git libvirt-dev libvirt-bin
# qemu-utils qemu-kvm -y
# wget https://releases.hashicorp.com/vagrant/1.8.1/vagrant_1.8.1_x86_64.deb
# sudo dpkg -i vagrant_1.8.1_x86_64.deb
#  vagrant box install boxcutter/ubuntu1404
#  vagrant plugin install vagrant-libvirt
#  vagrant plugin install vagrant-mutate
#  vagrant plugin install --plugin-version 0.0.3 fog-libvirt
#  vagrant mutate boxcutter/ubuntu1404 libvirt
###add your user to the appropriate libvirt or libvirtd group
#  sudo service libvirt-bin restart
#
server_box_name = "boxcutter/ubuntu1404"


# Uncomment the following lines with wbench_host objects and
# populate it with the correct MAC of the switch. Give each switch
# an IP ranging from 192.168.0.100-200. This is the dhcp pool provided
# by the VM.
#
# The example below shows 2 switches, leaf1 and leaf2 with their correct MAC
# address and IP assignments.
#
wbench_hosts = { :wbench_hosts => {} }

wbench_hosts[:wbench_hosts]['leaf1'] = {
 :ip => '192.168.0.100',
 :mac => '2C:60:0C:C0:11:11'}

wbench_hosts[:wbench_hosts]['leaf2'] = {
  :ip => '192.168.0.101',
  :mac => '2C:60:0C:C0:22:33'
 }
#
# The VM created runs DHCPD, BIND and a Web server.
#
# Bridge the eth1 interface directly to the physical interface leaving
# the laptop and connect the laptop nic interface to the mgmt interface
# of the switch. In my case, I did the following
#
# (on hypervisor) # virsh net-info switch_mgmt
# Name:           switch_mgmt
# UUID:           f9688f33-fce0-4ef6-b306-1348fa329617
# Active:         yes
# Persistent:     yes
# Autostart:      no
# Bridge:         virbr3 <=== attach phy interface to this bridge
#
# (on hypervisor) # virsh
# (on hypervisor) # brctl addif ens0p5 virbr3
#
# At no point is a console required, unless ONIE is messed up on the switch.
# which is rare
#
# Copy a Cumulus image to /var/www on the VM. The add a symlink for
# /var/www/onie-installer pointing to the cumulus image on /var/www on the VM.
# Example
#
# /var/www# ls
# lrwxrwxrwx  1 root    root           28 Apr 20 22:31 onie-installer -> CumulusLinux-2.5.7-amd64.bin
#
#
# Connect to the mgmt port of the switch and the switch will upgrade to the
# version of code specified by the onie-installer
#
# Wait 5 minutes. Unfortunately there is no easy way of telling if a switch has
# finished loading new OS. Just run a continuous ping of 192.168.0.100 from the
# VM after waiting 5 minutes. Then attempt to login using cumulus/CumulusLinux!
# as the SSH username and password
#
# Git clone a copy of the ansible playbook you wish to run into this env to the
# home directory of the cumulus user
#
# Run ``ansible -m setup all`` and it should execute successfully. After that
# execute the switch playbook. ``ansible-playbook lab_infra.yml --tags
# switches``
#
Vagrant.configure(2) do |config|

  # increase nic adapter count to be greater than 8
  # for all VMs.
  config.vm.provider :libvirt do |domain|
    domain.nic_adapter_count = 20
  end

  # vagrant issues #1673..fixes hang with configure_networks
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  config.vm.define :wbenchvm do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 256
    end
    node.vm.box = server_box_name
    # disabling sync folder support on all VMs
    node.vm.synced_folder '.', '/vagrant', :disabled => true

    # wbench_eth1
    node.vm.network :private_network,
      :ip => '192.168.0.1/24',
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'switch_mgmt'
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'ccw-wbenchvm-ansible/site.yml'
      ansible.extra_vars = wbench_hosts
      ansible.force_remote_user = false
    end
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'playbooks/wbenchvm_extra.yml'
    end
  end
end
