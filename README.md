# Kubernetes cluster with Vagrant

<details><summary><b>What is Kubernetes?</b></summary>


**Kubernetes (K8s)** is an open-source platform that is used to deploy and manage container. Below are some basic concepts in Kubernetes:

- **Node:** A server as cloud instance, VM of premise or computer where container can be deployed and run. There are 2 types:

    **Worker Node:**

    **Control Plan (Master) Node:**

- **Pod:** A group of containers deployed in the same **Node**. Each pod has a unique IP and shares network as well as storage resources to each other.

- **ReplicaSet:** This resource is used to manage a specify number of **Pods** are running for some purpose.
    
    *Example: If you want to have and persist 3 pods for your web app you have to define them in yaml/yml file.*

    <img src="/images/ReplicaSet.png" width=25% height=25%>

    *This yaml/yml file ensures your nginx app will always has 3 pods running **"replicas: 3"** in the same time with ReplicaSet resource.*        

- **Deployment:** This resource is used to deploy and manage **Pods** and **ReplicaSets**. It can update and rollback **Pods** via **ReplicaSet**.
    
    *Example: Your web app needs to have 2 versions. 1 is for lastest update and 1 is for backup version to rollback once it has accident. Therefore, you have to define 2 ReplicaSets in a Deployment yaml/yml file.*

    <img src="/images/Deployment.png" width=25% height=25%>

    *In this file, if you want to update app version, you just need to change the image then Kubenetes will create a new ReplicaSet with lastest image version for Pods.*

![](/images/Deployment_ReplicaSet_Pod.png)

- **StatefulSets** is a controller that is used to manage stability and consistency application. The best pactice is Database.

- **DaemonSets** is used to deploy an application pod to all node in cluster.

    *Example: You want to setup an Prometheus for all node to monitor. DaemonSets will help easily* 

</details>

<details><summary><b>What is Vagrant?</b></summary>

**Vagrant** is a tool to create infrastructure in virtual machine enviroment. It is **IaC** (Infrastructure as Code) on premise system that may help to define and manage virtual machines using code.

</details>

## Pre-required
- Use **Microsoft Windows 10** or **11**
- Use **Visual Studio Code** to interact with code as well as ``terminal`` or ``bash``.
- Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- Install [Vagrant](https://developer.hashicorp.com/vagrant/downloads)


#### Hyper-V issue

**Hyper-V** and its component make confliction to third party VM tools (VMWare, VirtualBox ...) so that **DO NOT** run or enable it. If your host machine enabled **Hyper-V**, refer these ways to disable it completely:
 
- Disable Hyper-V by uncheck in Windows features:

    <img src="/images/disable%20hyper%20v%20in%20windows%20features.png" width=50% height=50%>

- Run below command in PowerShell with administrator right then turn off and unplug the power as well as wait for 10s better than just reset your machine.

        bcdedit /set hypervisorlaunchtype off


## Getting started

### Provision VM in VirtualBox with Vagrant

<details><summary><b>Create Vagrantfile</b></summary>

Run `vagrant init` or create a file with *Vagrantfile* name.

Use as below code or [Vagrantfile](./Vagrantfile):

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

In this **Vagrantfile**, we simply specify:
- Number of virtual machines: ``NUM_MASTER_NODE``, ``NUM_WORKER_NODE``
- IP address: ``IP_NW``, ``MASTER_IP_START``, ``NODE_IP_START``
- Private networking connectivity: ``node.vm.network``
- Unique hostname: ``node.vm.hostname``
- Operating system: ``config.vm.box``
- System resources: ``vb.memory``, ``vb.cpus``
- GUI of VM Machine: `vb.gui`

**Vagrantfile** uses **Ruby** syntax. Refer [here](https://developer.hashicorp.com/vagrant/docs/vagrantfile) to get more information when modifying **Vagrantfile**.

</details>

<details><summary><b>Provisioning</b></summary>

1. Run this command

        vagrant up

    In this step, we may stuck when each machine is bootstrapped because of **Hyper-V** or hardware compatibility.

    ![](/images/stucking%20error.png)

    <img src="/images/stucking%20error%202.png" width=50% height=50%>

    If you do all ways in [Hyper-V issue](#hyper-v-issue) and still get this stucking:
    - Press "Enter" button to trigger manually from VM GUI. 
    - Increase boot_timeout (default is 300s) as terminal message in `Vagrantfile` (This is not the best practice to solve the issue).

        <img src="/images/increase%20boot_timeout.png" width=50% height=50%>
    
    - Remove the stucked-machine in **VirtualBox** and its resource in directory *"C:\Users\YourUser\VirtualBox VMs/"* then `vagrant up` again.
    - Re-install Windows OS (The last choice).

2. Verify provisioned-VM by command:

        vagrant status

    Result:

    <img src="/images/vagrant%20status.png" width=75% height=75%>

3. Remote to each node via ssh using command:

    Kubemaster

        vagrant ssh kubemaster

    Kubenode01

        vagrant ssh kubenode01

    Kubenode02

        vagrant ssh kubenode02



**Vagrant** fowards port 22 and generates keypair for `ssh` by itself so that we do not need to define them in `Vagrantfile`. Refer [Vagrant: SSH Sharing](https://developer.hashicorp.com/vagrant/docs/share/ssh) and [Vagrantfile: config.ssh](https://developer.hashicorp.com/vagrant/docs/vagrantfile/ssh_settings) for more information.

</details>

### Install Container Runtime (containerd) - All VM machines

**Container runtime** is a software to manage and run container in a system enviroment, it performs tasks such as creating, starting, stopping and deleting containers. There are some types of container runtime: CRI-O, Docker Engine, rkt (Rocket), LXC/LXD ... We will use *containerd* in this lab.

<details><summary><b>Load kernel modules in Linux</b></summary>

Run

    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF

    sudo modprobe overlay
    sudo modprobe br_netfilter

- Above commands help to define `overlay` and `br_netfilter` kernel module into k8s.conf file. `modules-load.d` is a system directory used for configuring the kernel module loading process with `.conf` file that specify the modules will be loaded when system boots up.

- `overlay` module is required when using Docker and Kubenetes because it can create a writeable layer on top of read-only image, allowing multiple containers to share the same image while still maintaining their own file systems.

- `br_netfilter` module that support to filter network packet during the network connection of the Linux kernel. Linux Bridge is a virtual network device that allows multiple physical or virtual network interfaces to be connected to each other to form a single network segment. The br_netfilter module is required to enable the use of iptables rules to filter network packets passing through the bridge. This is very important for containerization technologies like Docker and Kubernetes, as they use network bridges to connect containers to each other and to the outside world.

Verify

</details>

### Install kubeadm, kubelet and kubectl - All VM machines

### Bootstrap Control Plane and Worker nodes
