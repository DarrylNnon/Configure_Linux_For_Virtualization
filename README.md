# Configure Linux For Virtualization
Configuring Linux for virtualization involves setting up a hypervisor, managing virtual machines (VMs) from the command line, automating VM provisioning, and documenting the setup for future reference and team training. Below is a detailed, step-by-step guide for completing this project successfully using **KVM (Kernel-based Virtual Machine)** as the hypervisor. KVM is widely used in production environments for its performance and integration with Linux.

## Project Overview: Configure Linux for Virtualization

1. **Install and Configure KVM**: Set up KVM as the hypervisor for virtualization.
2. **Create and Manage VMs**: Use command-line tools like `virt-install` and `virsh` to manage virtual machines.
3. **Automate VM Provisioning**: Write scripts to create and configure VMs automatically.
4. **Document Virtualization Setup**: Document the process for easy replication and team training.


### Step 1: Install and Configure KVM

KVM is built into the Linux kernel and requires a CPU with virtualization extensions (Intel VT-x or AMD-V). We'll install KVM and supporting tools.

#### Step 1.1: Verify CPU Virtualization Support

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

- A value greater than `0` confirms that your CPU supports virtualization.

#### Step 1.2: Install KVM and Virtualization Tools

For **Ubuntu/Debian**:

```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
```

For **RHEL/CentOS/Fedora**:

```bash
sudo yum install -y qemu-kvm libvirt libvirt-python libvirt-client bridge-utils virt-install
```

#### Step 1.3: Start and Enable the Libvirt Service

```bash
sudo systemctl enable libvirtd
sudo systemctl start libvirtd
```

#### Step 1.4: Add Your User to the `libvirt` Group

This allows non-root users to manage VMs.

```bash
sudo usermod -aG libvirt $(whoami)
newgrp libvirt
```

#### Step 1.5: Verify Installation

Check if KVM is installed and running:

```bash
virsh list --all
```

- If this command returns no errors, KVM is installed correctly.

### Step 2: Create and Manage Virtual Machines with Command-Line Tools

We'll create and manage virtual machines using `virt-install` and `virsh`.

#### Step 2.1: Download an ISO Image for the Guest OS

Download an operating system ISO (e.g., Ubuntu Server):

```bash
wget https://releases.ubuntu.com/20.04/ubuntu-20.04.6-live-server-amd64.iso -P /var/lib/libvirt/images/
```

#### Step 2.2: Create a VM Using `virt-install`

Run the following command to create a VM:

```bash
sudo virt-install \
--name ubuntu-vm \
--ram 2048 \
--vcpus 2 \
--disk path=/var/lib/libvirt/images/ubuntu-vm.qcow2,size=20 \
--os-type linux \
--os-variant ubuntu20.04 \
--network bridge=virbr0,model=virtio \
--graphics none \
--cdrom /var/lib/libvirt/images/ubuntu-20.04.6-live-server-amd64.iso
```

- **Explanation**:
  - `--name`: Name of the VM.
  - `--ram`: Amount of memory allocated to the VM.
  - `--vcpus`: Number of virtual CPUs.
  - `--disk`: Disk size and path.
  - `--network`: Network configuration (using a virtual bridge here).
  - `--graphics none`: No graphical console (use SSH or terminal for management).
  - `--cdrom`: Path to the installation ISO.

#### Step 2.3: Manage VMs with `virsh`

- **List All VMs**:
  ```bash
  virsh list --all
  ```

- **Start a VM**:
  ```bash
  virsh start ubuntu-vm
  ```

- **Stop a VM**:
  ```bash
  virsh shutdown ubuntu-vm
  ```

- **Delete a VM**:
  ```bash
  virsh undefine ubuntu-vm --remove-all-storage
  ```

#### Step 2.4: Access the VM Console

Access the VM using `virsh console`:

```bash
virsh console ubuntu-vm
```

### Step 3: Automate VM Provisioning

Automation ensures faster and consistent VM creation. We'll create a script to provision VMs.

#### Step 3.1: Script for Automated VM Creation

**Script: `create_vm.sh`**

```bash
#!/bin/bash

# Check for root privileges
if [[ $EUID -ne 0 ]]; then
   echo "This script must be run as root"
   exit 1
fi

# Variables
VM_NAME=$1
RAM=$2
VCPUS=$3
DISK_SIZE=$4
ISO_PATH=$5

# Validate inputs
if [[ -z "$VM_NAME" || -z "$RAM" || -z "$VCPUS" || -z "$DISK_SIZE" || -z "$ISO_PATH" ]]; then
    echo "Usage: ./create_vm.sh <vm_name> <ram> <vcpus> <disk_size> <iso_path>"
    exit 1
fi

# Create the VM
virt-install \
--name "$VM_NAME" \
--ram "$RAM" \
--vcpus "$VCPUS" \
--disk path=/var/lib/libvirt/images/"$VM_NAME".qcow2,size="$DISK_SIZE" \
--os-type linux \
--os-variant ubuntu20.04 \
--network bridge=virbr0,model=virtio \
--graphics none \
--cdrom "$ISO_PATH" \
--noautoconsole

if [[ $? -eq 0 ]]; then
    echo "VM $VM_NAME created successfully."
else
    echo "Failed to create VM $VM_NAME."
fi
```

- **Execution**:
  ```bash
  chmod +x create_vm.sh
  sudo ./create_vm.sh ubuntu-vm-2 2048 2 20 /var/lib/libvirt/images/ubuntu-20.04.6-live-server-amd64.iso
  ```

#### Step 3.2: Automate Repetitive Tasks with Ansible (Optional)

For large-scale deployments, use **Ansible** to define and automate VM provisioning in an Infrastructure-as-Code (IaC) manner.


### Step 4: Document Virtualization Setup

Documenting the virtualization setup is critical for replication and team training. Use the following structure:

#### **Template: Virtualization Setup Document**

**1. Overview**  
This document outlines the virtualization setup using KVM, including installation, VM management, and automation.

**2. Environment Details**  
- Host OS: Ubuntu 22.04  
- Hypervisor: KVM  
- Tools: libvirt, virt-install, virsh  

**3. Setup Instructions**  
- Install KVM and necessary tools:
  ```bash
  sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
  ```
- Verify installation:
  ```bash
  virsh list --all
  ```

**4. VM Management**  
- Create VMs with `virt-install`:
  ```bash
  sudo virt-install --name ubuntu-vm --ram 2048 --vcpus 2 --disk path=/var/lib/libvirt/images/ubuntu-vm.qcow2,size=20 --os-type linux --os-variant ubuntu20.04 --cdrom /path/to/iso
  ```
- Manage VMs with `virsh`:
  ```bash
  virsh start ubuntu-vm
  virsh shutdown ubuntu-vm
  ```

**5. Automation**  
- Use `create_vm.sh` for automated VM creation.

**6. Troubleshooting**  
- Common issues:
  - **Virtualization not supported**: Ensure virtualization is enabled in BIOS/UEFI.
  - **Permission issues**: Ensure the user is in the `libvirt` group.


### Final Notes

1. **Testing**: Test all scripts and configurations in a staging environment before production use.
2. **Security**: Restrict access to VM management tools and enforce strong access controls on the host.
3. **Backup**: Regularly back up VMs using tools like `virt-backup`.

This step-by-step guide ensures a robust virtualization setup that is well-documented and easy for your team to learn and manage.
