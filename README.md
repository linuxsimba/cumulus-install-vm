# Provision Cumulus Switch from ONIE.


A simplification of
[vagrant-cw-libvirt](https://github.com/skamithi/vagrant-cw-libvirt). Received a bunch of Quanta switches
with no Cumulus software. This provided a quick way to install Cumulus Linux and
provision the switches with the appropriate config. No internet connectivity or
*console access* was needed during provisioning.

## Requirements

* Only supports
  [vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt), so only Linux Laptops.
  Support and documentation for
[vagrant-virtualbox](https://www.vagrantup.com/docs/providers/) is welcome, and then you
  can do this procedure using a MAC or Windows PC.



## Workflow

* Clone this repo

```
git clone https://github.com/linuxsimba/cumulus-install-vm --recursive
```

* Define ``wbench_hosts`` with the correct MAC addresses. Read the
  [Vagrantfile](https://github.com/linuxsimba/cumulus-install-vm/blob/master/Vagrantfile) for more details

* Create the VM using ``vagrant up --provider libvirt``. This will provision
  BIND, DHCPD, install ansible and position the cumulus user SSH key as the
  public key access to the root user of the switches.


* Copy a Cumulus Linux OS version to /var/www of the VM. Add a symlink called
  /var/www/onie-installer and point it to the Cumulus image

```
/var/www# ls
  lrwxrwxrwx  1 root  root   28 Apr 20 22:31 onie-installer -> CumulusLinux-2.5.7-amd64.bin

```

* Git clone the provision scripts to the Cumulus user home directory on the VM, if you want to
  completely configure the switch offline.

* Go into the lab, and stitch the physical port of the laptop to the mgmt port of the switch. Make sure to put the physical port and the VM's eth1 on the same software bridge.
  Example:
  ```
 (on laptop) # virsh net-info switch_mgmt
 Name:           switch_mgmt
 UUID:           f9688f33-fce0-4ef6-b306-1348fa329617
 Active:         yes
 Persistent:     yes
 Autostart:      no
 Bridge:         virbr3 <=== attach phy interface to this bridge

 (on laptop) # virsh
 (on laptop) # brctl addif ens0p5 virbr3

  ```

* reboot the switch.

* When the switch reboots, it comes up in ONIE mode and installs whatever OS is
  specified in the VM's /var/www/onie-installer path.

* It's not obvious when it's done, so I just waited 5 minutes then pinged
  associated IP. _See ``wbench_hosts`` of the [Vagrantfile](https://github.com/linuxsimba/cumulus-install-vm/blob/master/Vagrantfile)_. After that I confirmed I could ssh to
  the switch root user. Then I ran the Ansible provision script _(not included in
this repo)_ to configure the L2, L3 config and site specific apps.
  This was the longest step. Wish there was a quick way to
  simulate Ansible tower behaviour, which automates this step.

* Each switch install took about 10 minutes.

