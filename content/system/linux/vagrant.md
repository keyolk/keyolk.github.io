+++
title  = "vagrant"
toc    = true
weight = 0
+++

## Remote Vagrant Setup

#### Setup libvirtd

```
/usr/sbin/useradd -c 'HAL daemon' -u 68 -s /sbin/nologin -r -d '/' haldaemon
/usr/sbin/useradd -c 'dbus' -u 69 -s /sbin/nologin -r -d '/' dbus
mount -o ro,remount /sys; mount -o rw,remount /sys
mount -t mqueue none /dev/mqueue
service messagebus start
service libvirtd start
```

```
$ virsh -c qemu:///system list
```

#### Install Vagarnt on Client
```
wget https://releases.hashicorp.com/vagrant/1.9.1/vagrant_1.9.1_x86_64.rpm
rpm -ivh vagrant*.rpm
vagrant plugin install vagrant-libvirt --plugin-version 0.0.35

vagrant init fedora/24-cloud-base
vagrant up --provider=libvirt
```

#### Sample Vagrantfile
```
INSTANCE_PREFIX="centos"

$num_instance = 3
$box = "centos/7"

$vm_cpus = 2
$vm_memory = 1024

Vagrant.configure("2") do |config|
  def customize(config)
    config.vm.box = $box
    config.ssh.insert_key = false
    config.ssh.forward_agent = true
    config.ssh.forward_x11 = true

    config.vm.provider :libvirt do |libvirt|
      libvirt.driver = "kvm"
      libvirt.host = "sample"
      libvirt.username = "username"
      libvirt.storage_pool_name = "default"
      libvirt.connect_via_ssh = "true"
    end
  end

  $num_intance.times do |i|
    config.vm.define "#{INSTANCE_PREFIX}-#{i}.vagrant" do |target|
      customize target
      instance_index = i
      target.vm.hostname = "#{INSTANCE_PREFIX}-#{instance_index}.vagrant"
    end
  end
end
```
