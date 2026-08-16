# Provisioning Infrastructure with Vagrant and Libvirt/KVM (Bare Metal / Linux)

This guide walks you through setting up the required virtual machine infrastructure on a **Bare Metal / Linux Host (Debian/Ubuntu)** using **KVM**, **Libvirt**, and **Vagrant**.

## Prerequisites

Ensure your host system meets the following requirements:

* **OS:** Debian 11/12 or Ubuntu 20.04/22.04/24.04
* **Hardware:** Minimum 8GB RAM, 4 CPU cores, and Virtualization enabled in BIOS/UEFI (VT-x / AMD-V)



## Step 1: Install KVM, Libvirt, and Vagrant

1. Install KVM hypervisor packages and tools:

   ```bash
   sudo apt update
   sudo apt install -y qemu-system-x86 libvirt-daemon-system libvirt-clients virtinst ruby-dev libvirt-dev build-essential
   ```

2. Add your current user to the `libvirt` and `kvm` groups so you can manage VMs without `sudo`:

   ```bash
   sudo usermod -aG libvirt,kvm $USER
   newgrp libvirt
   ```

3. Install Vagrant and the `vagrant-libvirt` provider plugin:

   ```bash
   sudo apt install -y vagrant
   vagrant plugin install vagrant-libvirt
   ```



## Step 2: Configure System Settings

1. **Disable Swap:** Kubernetes requires swap space to be disabled across all nodes and the host:

   ```bash
   sudo swapoff -a
   sudo sed -i '/swap/d' /etc/fstab
   ```

2. **Fix Firewalld Conflict (If applicable):** If using `firewalld`, switch its backend to `iptables` to ensure `libvirt` network creation succeeds:

   ```bash
   sudo sed -i 's/FirewallBackend=nftables/FirewallBackend=iptables/g' /etc/firewalld/firewalld.conf
   sudo systemctl restart firewalld
   sudo systemctl restart libvirtd
   ```



## Step 3: Configure the Vagrantfile

Ensure your `vagrant/Vagrantfile` uses a base box that supports the `libvirt` provider (such as `generic/ubuntu2204`):

```ruby
config.vm.box = "generic/ubuntu2204"
```



## Step 4: Spin Up the Infrastructure

Navigate to the `vagrant` directory and start the cluster nodes:

```bash
cd vagrant
vagrant up --provider=libvirt
```

This will launch 5 virtual machines:

* `controlplane01`
* `controlplane02`
* `loadbalancer`
* `node01`
* `node02`



## Step 5: Verify the Cluster State

1. Check the status of all VMs:

   ```bash
   vagrant status
   ```

2. Verify connectivity into a node:

   ```bash
   vagrant ssh controlplane01
   ```
