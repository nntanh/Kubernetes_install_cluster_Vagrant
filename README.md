# Create Kubernetes cluster with Vagrant
**Vagrant** is a tool to create infrastructure in virtual machine enviroment. It is IaC (Infrastructure as Code) on premise system that may help to define and manage virtual machines using code.

## Pre-required
- Use **Microsoft Windows 10** or **11**
- Use **Visual Studio Code** to interact code as well as ``terminal`` or ``bash``.
- Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- Install [Vagrant](https://developer.hashicorp.com/vagrant/downloads)


#### Hyper-V issue

**Hyper-V** and its component make confliction to third party VM tools (VMWare, VirtualBox ...) so that **DO NOT** run or enable it. If your host machine enabled **Hyper-V**, refer these ways to disable it completely:
 
- Disable Hyper-V by uncheck in Windows features:

![disable hyper v](/doc/images/disable%20hyper%20v%20in%20windows%20features.png)

- Run below command in PowerShell with administrator right then turning off and unplugging the power are better reset your machine.

        bcdedit /set hypervisorlaunchtype off

## Getting started

### Provision VM in VirtualBox with Vagrant
<details><summary><b>Create Vagrantfile</b></summary>

1. Use as below code:

```
# -*- mode: ruby -*-
# vi:set ft=ruby sw=2 ts=2 sts=2:

# Define the number of control plane (MASTER_NODE) and node (WORKER_NODE)
NUM_MASTER_NODE = 1
NUM_WORKER_NODE = 2

IP_NW = "192.168.56."
MASTER_IP_START = 1
NODE_IP_START = 2

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  # Here are some key details about the "ubuntu/bionic64" Vagrant box:
    # Operating System: Ubuntu 18.04 LTS (Bionic Beaver)
        # Ubuntu 18.04 LTS will receive security updates and bug fixes 
        # from Canonical, the company behind Ubuntu, until April 2023 
        # for desktop and server versions, and until April 2028 for 
        # server versions with Extended Security Maintenance (ESM) enabled.
    # Architecture: x86_64 (64-bit)
    # Disk Size: 10 GB
    # RAM: 2 GB
    # CPUs: 2
    # Desktop Environment: None (headless)
    # Provider: VirtualBox
  config.vm.box = "ubuntu/bionic64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

  # View the documentation for the VirtualBox for more
  # information on available options.
  # https://developer.hashicorp.com/vagrant/docs/providers/virtualbox/configuration

  # Provision Control Plane
  (1..NUM_MASTER_NODE).each do |i|
      config.vm.define "kubemaster" do |node|
        node.vm.provider "virtualbox" do |vb|
            vb.name = "kubemaster"
            vb.memory = 2048
            vb.cpus = 2
        end
        node.vm.hostname = "kubemaster"
        node.vm.network :private_network, ip: IP_NW + "#{MASTER_IP_START + i}"
      end
  end


  # Provision Nodes
  (1..NUM_WORKER_NODE).each do |i|
    config.vm.define "kubenode0#{i}" do |node|
        node.vm.provider "virtualbox" do |vb|
            vb.name = "kubenode0#{i}"
            vb.memory = 2048
            vb.cpus = 2
        end
        node.vm.hostname = "kubenode0#{i}"
        node.vm.network :private_network, ip: IP_NW + "#{NODE_IP_START + i}"
    end
  end
end
```

2. Another way, use [Vagrantfile](./Vagrantfile)

In this **Vagrantfile**, we simply specify:
- Number of virtual machines: ``NUM_MASTER_NODE``, ``NUM_WORKER_NODE``
- IP address: ``IP_NW``, ``MASTER_IP_START``, ``NODE_IP_START``
- Private networking connectivity: ``node.vm.network``
- Unique hostname: ``node.vm.hostname``
- Operating system: ``config.vm.box``
- System resources: ``vb.memory, vb.cpus``
- Open GUI of VM Machine: `vb.gui`

**Vagrantfile** use **Ruby** syntax. Refer [here](https://developer.hashicorp.com/vagrant/docs/vagrantfile) to get more information when modifying **Vagrantfile**.

</details>

<details><summary><b>Provisioning</b></summary>

Run this command

    vagrant up

##### Bootstrap stucking error
In this step, we may stuck when each machine is bootstrapped because of **Hyper-V**.

![stucking error](/doc/images/stucking%20error.png)


If you do all ways in [Hyper-V issue](#hyper-v-issue) and still get this stucking, press "Enter" button to trigger manually from VM GUI.


</details>

### Install Container Runtime (containerd) - All VM machines

### Install kubeadm, kubelet and kubectl - All VM machines

### Bootstrap Control Plane and Worker nodes
