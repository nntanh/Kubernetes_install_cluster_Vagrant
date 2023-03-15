# Create Kubernetes cluster with Vagrant
**Vagrant** is a tool to create infrastructure in virtual machine enviroment. It is IaC (Infrastructure as Code) on premise system that may help to define and manage virtual machines using code.

## Pre-required
- Using **Microsoft Windows 10** or **11**
- Using **Visual Studio Code** to interact code as well as terminal or bash.
- Install **VirtualBox** https://www.virtualbox.org/wiki/Downloads
`This lab uses VirtualBox for Virtual Machine`
- Install **Vagrant** https://developer.hashicorp.com/vagrant/downloads

**Note:** Do not run or install Hyper-V and any its component because it conflicts with third party virtualization tools (VMWare, VirtualBox ...).
If your host machine enabled Hyper-V, run `bcdedit /set hypervisorlaunchtype off` command in PowerShell with administration right. This is not sure to disable Hyper-V clearly so that the best practice is re-install Windows or press `enter button` when facing the VM bootstrapping by `vagrant up`
 
## Getting started
- [Provision VM in VirtualBox with Vagrant](#provision-vm-in-virtualbox-with-vagrant)
- [Install Container Runtime (containerd) - All VM machines](#install-container-runtime-containerd---all-vm-machines)
- [Install kubeadm, kubelet and kubectl - All VM machines](#install-kubeadm-kubelet-and-kubectl---all-vm-machines)
- [Bootstrap Control Plane and Worker nodes](#bootstrap-control-plane-and-worker-nodes)

### Provision VM in VirtualBox with Vagrant
- Create Vagrantfile as below/

### Install Container Runtime (containerd) - All VM machines

### Install kubeadm, kubelet and kubectl - All VM machines

### Bootstrap Control Plane and Worker nodes