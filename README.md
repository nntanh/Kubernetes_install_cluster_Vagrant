# Kubernetes cluster with Vagrant

<details><summary><b>What is Kubernetes?</b></summary>


**Kubernetes (K8s)** is an open-source platform that is used to deploy and manage container (Container runtime). Below are some basic concepts in Kubernetes:

- **Node** is a server such as cloud instance, VM of premise or computer where container can be deployed and run. There are 2 types:

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

**Vagrant** is a tool to create infrastructure in virtual machine enviroment. It is **IaC** (Infrastructure as Code) on premise system that may help to define and manage virtual machines using code. Refer [Introduction to Vagrant](https://developer.hashicorp.com/vagrant/intro) to get more information.

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

<details><summary><b>Provision Vagrant</b></summary>

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

### Install Container Runtime (containerd) - All nodes

**Container runtime** is a software to manage and run container in a system enviroment, it performs tasks such as creating, starting, stopping and deleting containers. There are some types of container runtime: *CRI-O*, *Docker Engine*, *containerd*, *rkt (Rocket)*, *LXC/LXD*, *K8s* ... We use *containerd* in this lab.

<details><summary><b>Load kernel modules in Linux</b></summary>

Run

    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF

    sudo modprobe overlay
    sudo modprobe br_netfilter

- Above commands help to define `overlay` and `br_netfilter` kernel module into `k8s.conf` file. `modules-load.d` is a system directory for configuring the kernel module loading process with `.conf` file that specifies the modules will be loaded when system boots up.

- `overlay` module is required when using Docker and Kubenetes because it can create a writeable layer on top of read-only image, allowing multiple containers to share the same image while still maintaining their own file systems.

- `br_netfilter` module supports to filter network packet during the network connection of the Linux kernel. Linux Bridge is a virtual network device that allows multiple physical or virtual network interfaces to be connected to each other to form a single network segment. The br_netfilter module is required to enable the use of iptables rules to filter network packets passing through the bridge. This is very important for containerization technologies like Docker and Kubernetes, as they use network bridges to connect containers to each other and to the outside world.

Verify command:

    lsmod | grep overlay
    lsmod | grep br_netfilter

</details>

<details><summary><b>Setup iptables to handle network packet</b></summary>

Run

    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF

    sudo sysctl --system

- `net.bridge.bridge-nf-call-iptables  = 1` command enables to allow packets pass through network bridges by iptables. These packets will be sent to the FORWARD chain of iptables for further control.

- `net.ipv4.ip_forward = 1` command enables to foward the packets between different network interfaces and their intended destination address on the system. This is an important feature for implementing complex network solutions such as virtualization or distributed computer networks.

- `sudo sysctl --system` apply above setting without reboot.

Verify command

    sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward

</details>

<details><summary><b>Install containerd</b></summary>

In this lab, we use `containerd.io` that does not contain **CNI** plugins (install later when bootstrapping control plane and worker nodes).

>CNI (Container Network Interface) has some plugins to support brigde network - iptables such as Flannel, Calico, Weave net (These plugins need to use iptables to config firewall and routing). CNI is compatible with many different network technologies, allowing integration and expansion of network technologies as needed for each application.

Run

    sudo apt-get update

    sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

The above command is used to install necessary packages to ensure security and authentication when using other commands in Ubuntu or Debian-based systems. The packages include:

- ``ca-certificates`` contains necessary public certificates to authenticate connections to online services.
- ``curl`` a command-line tool to transfer data to or from a server.
- ``gnupg`` a complete and free implementation of the OpenPGP standard, used for encrypting and signing data.
- ``lsb-release`` provides information about the version of the distribution being used.

Add Docker’s official GPG key

    sudo mkdir -m 0755 -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

Set up the repository

    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

Install ``containerd.io``

    sudo apt-get update
    sudo apt-get install containerd.io

</details>

<details><summary><b>Install cgroup (Control group)</b></summary>

**Cgroup (Control group)** is a feature in the Linux operating system that allows for the management and limitation of system resources such as CPU, memory, I/O, and networking for a specific group of processes. These process groups can be related to an application or containerization systems like Docker or Kubernetes.

When installing **cgroup** for a **Kubernetes (k8s) cluster**, the cluster's resources will be managed and limited by **cgroup** to ensure system performance and stability. However, **K8s** does not require users to actively divide resources for processes because it uses abstract objects such as Pods and Containers to manage resources instead of creating and configuring **cgroups** for each application. Therefore, **K8s** will use **cgroups** to manage and limit resources for these Pods and containers, ensuring that they do not affect other processes running on the same node.

There are 2 **cgroup** drivers: **cgroupfs** and **systemd**. 

In this lab, we have `kubelet` and `containerd` using **systemd**. You can use to check the **cgroup** driver type by command:

    cat /proc/mounts | grep cgroup

Run below command to config ``containerd`` use systemd:

    sudo vi /etc/containerd/config.toml

Replace all with below content:

    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true

Restart ``containerd``

    sudo systemctl restart containerd

</details>

### Install kubeadm, kubelet and kubectl - All nodes

- **Kubeadm** is a command-line tool for installing and setting up a Kubernetes cluster. Kubeadm provides a standardized way to create and manage a Kubernetes cluster, including installing worker nodes and control plane nodes.

- **Kubelet** is a component on each worker node in a Kubernetes cluster, responsible for managing containers on that node. It works with Kubernetes API servers to synchronize the state of containers and other related objects.

- **Kubectl** is a command-line tool for interacting with the Kubernetes API server. Kubectl allows users to create, modify, and delete objects in the cluster, such as pods, services, deployments, and other objects. Kubectl also provides information about the status of objects in the cluster.

<details><summary><b>Disable swap</b></summary>

**Swap** is a partition on the hard drive of a Linux system used to reduce the use of RAM memory. When the system's RAM is full, swap is used to store data and processes that are currently not being used much to free up RAM memory for other processes.

**Pod** and **container** will run as process in nodes (K8s cluster) and they have to be stability and reliability for application without any impact from another function so that **swap** should be disabled in K8s.

Run

    # First diasbale swap
    sudo swapoff -a

    # And then to disable swap on startup in /etc/fstab
    sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

</details>

<details><summary><b>Install kubeadm, kubelet and kubectl packages</b></summary>

``kubeadm``, ``kubelet`` and ``kubectl`` need to match with Kubernetes control plane. Refer [here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#version-skew-policy) to get more information.

Setup **ca-certificate** package for secure communication with **HTTPS-based repositories** using curl.

    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates curl

Get **Google Cloud public signing key**.

    sudo mkdir -m 0755 -p /etc/apt/keyrings
    sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

Add **Kubernetes apt repository**.

    echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

Install `kubelet`, `kubeadm` and `kubectl` then pin their version to avoid auto-upgrade:

    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl

*The client certificates generated by kubeadm will be expired after 1 year. Refer [here](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/) to get more information*

</details>

### Bootstrap Control Plane and Worker nodes

<details><summary><b>Init Control Plane (kubemaster)</b></summary>

Run command:

    sudo kubeadm init --apiserver-advertise-address=192.168.56.2 --pod-network-cidr=10.244.0.0/16

- ``--apiserver-advertise-address=192.168.56.2``: Advertise API server of cluster. In this case, we use the same with master node.

- `--pod-network-cidr=10.244.0.0/16`: Create CIDR for pod network. If not, the master node will generate pod CIDR by itself. Do not chose the exist range to avoid confliction.

Success message:

    Your Kubernetes control-plane has initialized successfully!

    To start using your cluster, you need to run the following as a regular user:

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

    You should now deploy a Pod network to the cluster.
    Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
    /docs/concepts/cluster-administration/addons/

    You can now join any number of machines by running the following on each node
    as root:

    kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>

Save ``kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>`` to join worker node later.

Make `kubectl` run with **non-root user**.

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

If you just want to use **root user** for work, run below command.

    export KUBECONFIG=/etc/kubernetes/admin.conf

</details>

<details><summary><b>Join Worker node (kubenode01, kubenode02)</b></summary>

Use the command copied.

    sudo kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>

You can get the token again by command in **kubemaster**.

    kubeadm token list

Output:

        TOKEN                    TTL  EXPIRES              USAGES           DESCRIPTION            EXTRA GROUPS
    8ewj1p.9r9hcjoqgajrj4gi  23h  2018-06-12T02:51:28Z authentication,  The default bootstrap  system:
                                                    signing          token generated by     bootstrappers:
                                                                        'kubeadm init'.        kubeadm:
                                                                                            default-node-token

Token will be expired after 24 hours by default. Run below command in **kubemaster** to get new token.

    kubeadm token create

Run below command to get <hash>.

    openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
    openssl dgst -sha256 -hex | sed 's/^.* //'

Output.

    8cb2de97839780a412b93877f8507ad6c94f73add17d5d7058e91741c9d5ec78

Success message.

    [preflight] Running pre-flight checks

    ... (log output of join workflow) ...

    Node join complete:
    * Certificate signing request sent to control-plane and response
    received.
    * Kubelet informed of new secure connection details.

    Run 'kubectl get nodes' on control-plane to see this machine join.

</details>

<details><summary><b>Verify Kubernetes cluster components</b></summary>

Run below command on all nodes.

    sudo netstat -lntp

Kubemaster output:

    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
    tcp        0      0 127.0.0.1:10248         0.0.0.0:*               LISTEN      8013/kubelet
    tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      8182/kube-proxy
    tcp        0      0 192.168.56.2:2379       0.0.0.0:*               LISTEN      7811/etcd
    tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      7811/etcd
    tcp        0      0 192.168.56.2:2380       0.0.0.0:*               LISTEN      7811/etcd
    tcp        0      0 127.0.0.1:2381          0.0.0.0:*               LISTEN      7811/etcd
    tcp        0      0 127.0.0.1:10257         0.0.0.0:*               LISTEN      7791/kube-controlle
    tcp        0      0 127.0.0.1:10259         0.0.0.0:*               LISTEN      7907/kube-scheduler
    tcp        0      0 127.0.0.1:34677         0.0.0.0:*               LISTEN      2826/containerd
    tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      817/systemd-resolve
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1380/sshd
    tcp6       0      0 :::10250                :::*                    LISTEN      8013/kubelet
    tcp6       0      0 :::6443                 :::*                    LISTEN      7884/kube-apiserver
    tcp6       0      0 :::10256                :::*                    LISTEN      8182/kube-proxy
    tcp6       0      0 :::22                   :::*                    LISTEN      1380/sshd

Kubenode output:

    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    tcp        0      0 127.0.0.1:10248         0.0.0.0:*               LISTEN      8987/kubelet        
    tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      9208/kube-proxy     
    tcp        0      0 127.0.0.1:39989         0.0.0.0:*               LISTEN      2785/containerd     
    tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      782/systemd-resolve 
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1431/sshd
    tcp6       0      0 :::10250                :::*                    LISTEN      8987/kubelet        
    tcp6       0      0 :::10256                :::*                    LISTEN      9208/kube-proxy     
    tcp6       0      0 :::22                   :::*                    LISTEN      1431/sshd

</details>

<details><summary><b>Install Pod network add-on</b></summary>

Run below command to check all system nodes.

    kubectl get nodes

Output:

    NAME           STATUS     ROLES           AGE    VERSION
    kubemaster     NotReady   control-plane   3h1m   v1.26.2
    kubenode01     NotReady   <none>          3h     v1.26.2
    kubenode02     NotReady   <none>          179m   v1.26.2

Their status is `NotReady`. Run ``kubectl get pods -A`` to check.

    NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
    kube-system   coredns-787d4945fb-5cwlq               0/1     Pending   0          3h8m
    kube-system   coredns-787d4945fb-q2s4p               0/1     Pending   0          3h8m
    kube-system   etcd-controlplane                      1/1     Running   0          3h8m
    kube-system   kube-apiserver-controlplane            1/1     Running   0          3h8m
    kube-system   kube-controller-manager-controlplane   1/1     Running   0          3h8m
    kube-system   kube-proxy-7twwr                       1/1     Running   0          3h7m
    kube-system   kube-proxy-8mxt7                       1/1     Running   0          3h8m
    kube-system   kube-proxy-v9rc6                       1/1     Running   0          3h8m
    kube-system   kube-scheduler-controlplane            1/1     Running   0          3h9m

**CoreDNS** (Cluster DNS) is not started up because **pod network add-on** is not installed completely. It must have **CNI** (Container network interface) plugin to run and manage network for cluster.

In this lab, we will use [Weave Net](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/) add-on.

Run below command in `kubemaster`

    kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

Output:

    serviceaccount/weave-net created
    clusterrole.rbac.authorization.k8s.io/weave-net created
    clusterrolebinding.rbac.authorization.k8s.io/weave-net created
    role.rbac.authorization.k8s.io/weave-net created
    rolebinding.rbac.authorization.k8s.io/weave-net created
    daemonset.apps/weave-net created

Open DaemonSet yaml file in kubemaster:

    kubectl edit ds weave-net -n kube-system

Add ``IPALLOC_RANGE`` variable and set value of ``--pod-network-cidr``

    spec:
    ...
        template:
        ...
            spec:
            ...
                containers:
                ...
                    env:
                    - name: IPALLOC_RANGE
                      value: 10.244.0.0/16
                      ...
                    name: weave

Save file then wait a few minutes to reboot.

Check `kubectl get pods -A` and `kubectl get nodes` again for the success result.

</details>