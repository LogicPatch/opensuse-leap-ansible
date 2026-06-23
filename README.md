![](logo.png)

# 🦎 OpenSuSE Leap Ansible

Automated openSUSE Leap setup and customization using Ansible playbooks.

---

# 🐧 Overview

This project provides a collection of Ansible playbooks to automate the installation and configuration of openSUSE Leap systems.

It helps quickly configure a fresh openSUSE installation with:

- additional repositories
- system updates
- hardware drivers
- desktop customization
- productivity tools
- development tools
- multimedia software
- common desktop applications

The project is designed to create a reproducible and modular openSUSE desktop environment.

---

# 🎯 Who is this for?

- openSUSE users reinstalling systems frequently
- Linux enthusiasts building reproducible workstations
- Developers setting up new machines
- Users migrating to new hardware
- Anyone wanting automated desktop provisioning

---

# 💡 Key Features

- ✅ Ansible-based automation
- ✅ Reproducible installations
- ✅ Modular playbook structure
- ✅ openSUSE Leap focused
- ✅ SSH-based deployment
- ✅ Host-only lab environment tested
- ✅ GNOME customization
- ✅ Driver installation
- ✅ Desktop application provisioning
- ✅ Easy to extend and maintain

---

# 🏗️ Test Environment

## Host System

- NixOS
- Ansible Core 2.x
- KVM / Libvirt

## Target System

- openSUSE Leap 16

---

# 🌐 Host-Only Network Setup (KVM / libvirt)

This project uses a **host-only network** to provide stable SSH access between the NixOS host and the openSUSE guest VM.

It is isolated from the external network but still reachable from the host system.



## Network Design

| Component     | IP Address      | Role                |
|--------------|----------------|---------------------|
| NixOS Host   | 192.168.56.1   | Ansible Controller  |
| openSUSE VM  | 192.168.56.10  | Managed Node        |

---

## libvirt Network Definition

The host-only network is defined in:
`hostonly.xml`  
SSH connectivity is used for all Ansible deployments.


### Create and Enable Network

Define the network:
```bash
sudo virsh net-define hostonly.xml
```

Start the network:
```bash
sudo virsh net-start hostonly
```

Enable autostart:
```bash
sudo virsh net-autostart hostonly
```

Verify status:
```bash
sudo virsh net-list --all
```

### Verify Host Interface

After starting the network, verify that the bridge interface has been created correctly:

```bash
ip addr show virbr1
```

Expected result:

- The interface `virbr1` exists  
- It is assigned the IP address `192.168.56.1`  
- The state should be `UP` (or at least not missing)

---

## VM Network Configuration

In the virtual machine configuration (virt-manager), the network adapter must be set to the host-only libvirt network to ensure SSH connectivity between host and guest.


### Network Source

Select:
In the virt-manager main window:  
**Double-click the relevant VM  →  Show virtual hardware details  →  Add hardware  →  Select "Network" in the left column**  
- **Network source:** `Virtual Network 'hostonly' (isolated network)`  

This ensures the VM is attached to the libvirt-managed host-only bridge.  

Recommended settings:  
- **Device model:** `virtio` (for best performance)  
- **MAC address:** can be left automatic or set manually for reproducibility  
- Leaving the MAC address on **auto-generated** is fine for testing  
- For reproducible lab setups (e.g. Ansible inventory consistency), a **fixed MAC address is recommended**  
- Changing the MAC will usually result in a new DHCP lease from libvirt

### After booting the VM (openSUSE)
Network settings via NetworkManager  
`nmtui`  
Then	→	`Edit a connection`  
Select the host-only network interface (e.g., `Wired connection 2`  `<Edit>`)  

Profile name		→	`Hostonly`  
IPv4-Configuration	→	`<manually>	<Show>`  
Adresses			→	`192.168.56.10/24`  
Gateway				→	leave blank ( important! )  
DNS					→	Optional
					→	`<OK>`  

### Start SSH daemon
```
sudo systemctl start sshd
sudo systemctl enable sshd
```  



### Expected Result

After booting the VM:

- The VM receives an IP address in the host-only range (e.g. `192.168.56.10`)
- The host can ping the VM
- SSH access is available from host to guest

Example:

```bash
ping 192.168.56.10
ssh mastermind@192.168.56.10
```

# 🔐 SSH Configuration

The target system is managed using SSH key authentication.

Generate and copy key:

```bash
ssh-keygen -t ed25519
ssh-copy-id mastermind@192.168.56.10
ssh mastermind@192.168.56.10
```


# 🚀 Ansible Test

Verify Ansible connectivity:

```bash
ansible all -m ping
ansible opensuse -m ping
ansible suse -m ping
```

Expected output:

```text
suse | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
If this succeeds, the environment is fully ready for automation.


---


# 📋 Recommended Execution Flow

The playbooks should be executed in the following order:

1. repositories.yaml  
2. base-system.yaml  
3. gnome.yaml  
4. applications.yaml  


Run full setup:
```bash
ansible-playbook site.yaml
```

Or run individual components:
```bash
ansible-playbook playbooks/repositories.yaml
ansible-playbook playbooks/base-system.yaml
ansible-playbook playbooks/gnome.yaml
ansible-playbook playbooks/applications.yaml
```


# 🚧 Notes

- This setup is intended for a controlled lab environment
- SSH key authentication is required for unattended execution
- Inventory should be adapted for each target system
- Playbooks are designed to be idempotent

---

# ⚠️ Disclaimer

This project modifies system configuration, installed packages, and desktop settings.

Always review playbooks before execution.

Use at your own risk.

---

# 📜 License

MIT License

