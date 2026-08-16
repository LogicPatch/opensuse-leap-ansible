![](logo.png)

# 🦎 openSUSE Leap Ansible

Automated provisioning and configuration of **openSUSE Leap** workstations using **Ansible**.

This project provides a modular collection of Ansible playbooks that transforms a fresh openSUSE Leap installation into a fully configured desktop and development workstation.

The playbooks are designed to be **reproducible**, **idempotent**, and **easy to customize**, allowing the same configuration to be applied repeatedly without unwanted side effects.

---

# 🐧 Overview

Setting up a new Linux installation often requires installing hundreds of packages, enabling repositories, configuring desktop settings, installing development tools, and customizing the system.

This project automates these repetitive tasks.

The goal is to create a reproducible openSUSE Leap environment that can be deployed within minutes instead of hours.

The project currently supports:

- Native openSUSE packages (`zypper`)
- Flatpak applications
- Third-party repositories
- Stand-alone RPM packages
- GNOME desktop customization
- Development environments
- Virtualization software
- Multimedia applications
- Gaming software
- Hardware configuration
- Modular feature selection through Ansible variables

---

# 🎯 Who is this project for?

This project is intended for:

- openSUSE Leap users
- Linux enthusiasts
- Developers
- DevOps engineers
- Homelab users
- Anyone who frequently reinstalls Linux
- Anyone wanting reproducible workstation provisioning

---

# ✨ Features

- ✅ Fully Ansible-based automation
- ✅ Modular playbook structure
- ✅ Idempotent execution
- ✅ Reproducible installations
- ✅ SSH-based deployment
- ✅ openSUSE Leap focused
- ✅ Flatpak integration
- ✅ Third-party repository management
- ✅ Stand-alone RPM installation support
- ✅ GNOME desktop customization
- ✅ Development workstation provisioning
- ✅ Multimedia workstation provisioning
- ✅ Gaming software installation
- ✅ Easy to extend

---

# 🏗 Project Architecture

The project is organized into independent modules.

```text
site.yaml
│
├── repositories.yaml
├── base-system.yaml
├── gnome.yaml
└── applications.yaml
      ├── applications_zypper.yaml
      ├── applications_flatpak.yaml
      ├── applications_repository.yaml
      └── applications_rpm.yaml
```

Each module is responsible for one clearly defined task.

This keeps the project maintainable and allows individual components to be executed independently.

---

# 📦 Supported Installation Methods

Applications can be installed using four different mechanisms.

| Method | Description |
|---------|-------------|
| **zypper** | Native openSUSE packages |
| **Flatpak** | Applications from Flathub |
| **Repository** | Packages requiring an additional third-party repository |
| **RPM** | Stand-alone RPM packages downloaded directly from vendors |

This abstraction allows every application to use the most appropriate installation method while keeping the playbooks generic.

---

# ⚙ Configuration

The project is controlled almost entirely through Ansible variables.

The primary configuration is stored in:

```text
group_vars/
└── opensuse.yaml
```

Applications can be enabled or disabled individually.

Example:

```yaml
applications:
  firefox: true
  brave: true
  signal_desktop: true
  steam: false
  wine: true
```

Only enabled applications are processed during playbook execution.

---

# 🧪 Test Environment

## Host System

- NixOS
- Ansible Core
- KVM / libvirt
- Ansible Controller

## Target System

- openSUSE Leap 16
- GNOME Desktop
- OpenSSH Server

The project is primarily developed and tested inside virtual machines
before being deployed on physical hardware.

The KVM/libvirt environment is used exclusively as a test environment
and is not required for using the playbooks on a physical system.

---

# 🔐 SSH Configuration

The target machine is managed using SSH public key authentication.

Generate a key pair:

```bash
ssh-keygen -t ed25519
```

Copy the key:

```bash
ssh-copy-id mastermind@192.168.56.10
```

Verify access:

```bash
ssh mastermind@192.168.56.10
```

Once SSH authentication works, the system is ready for Ansible automation.

---

# 🚀 Running the Playbooks

The recommended way to configure a fresh installation is to execute the complete site playbook.

```bash
ansible-playbook site.yaml -K
```

The `-K` option (`--ask-become-pass`) prompts for the sudo password required for privileged system tasks. The password is requested once at the beginning of the playbook execution
and is used for tasks requiring privilege escalation.

For unattended execution, an alternative Ansible become configuration can be used instead.

---


# 📋 Playbook Execution Order

When running the playbooks individually, the recommended execution order is:

1. `repositories.yaml`
2. `base-system.yaml`
3. `gnome.yaml`
4. `applications.yaml`

Examples:

```bash
ansible-playbook playbooks/repositories.yaml -K
```

```bash
ansible-playbook playbooks/base-system.yaml -K
```

```bash
ansible-playbook playbooks/gnome.yaml -K
```

```bash
ansible-playbook playbooks/applications.yaml -K
```

---

# 🏷️ Available Tags

Individual parts of the project can be executed independently using Ansible tags.

## Configure repositories

```bash
ansible-playbook playbooks/repositories.yaml --tags repositories -K
```

## Configure the base system

```bash
ansible-playbook playbooks/base-system.yaml --tags base_system -K
```

## Configure the GNOME desktop

```bash
ansible-playbook playbooks/gnome.yaml --tags gnome -K
```

## Install all applications

```bash
ansible-playbook playbooks/applications.yaml --tags applications -K
```

## Install applications from openSUSE repositories

```bash
ansible-playbook playbooks/applications.yaml --tags applications_zypper -K
```

## Install Flatpak applications

```bash
ansible-playbook playbooks/applications.yaml --tags applications_flatpak -K
```

## Configure third-party repositories and install related applications

```bash
ansible-playbook playbooks/applications.yaml --tags applications_repository -K
```

## Install standalone RPM packages

```bash
ansible-playbook playbooks/applications.yaml --tags applications_rpm -K
```


Tags can be combined when required:
```bash
ansible-playbook playbooks/applications.yaml --tags applications_zypper,applications_flatpak -K
```

---

# 📂 Repository Strategy
This project supports several repository types.

| Repository | Purpose | Recommended |
|---|---|---|
| OSS | Base system packages | ✔ Always enabled |
| Packman | Multimedia codecs and packages | ✔ Recommended |
| libdvdcss | DVD playback support | Optional |
| NVIDIA | Proprietary NVIDIA drivers | Optional |

## Default openSUSE repositories

The standard Leap repositories provide the majority of desktop packages.

Examples include:

- Firefox
- Thunderbird
- GNOME
- KDE
- Development libraries

## Packman Repository

Packman provides multimedia packages that cannot be shipped with the default distribution.

Typical packages include:

- VLC codecs
- FFmpeg
- GStreamer plugins

## NVIDIA Repository

The NVIDIA repository provides proprietary NVIDIA graphics drivers.

The repository is added automatically when NVIDIA support is enabled in
the Ansible configuration.

Enable it only on systems equipped with NVIDIA graphics hardware.

## libdvdcss Repository

Optional repository providing `libdvdcss` for DVD playback support.

## Application-specific Repositories

Some applications require their own upstream repositories.

Currently supported:

- Google Chrome
- Wine

The required repositories are automatically added and refreshed by the corresponding application playbooks.

---

# 🔄 Package Installation Strategy

Applications are installed using the most appropriate method.

| Application | Installation Method |
|------------|---------------------|
| Firefox | zypper |
| Thunderbird | zypper |
| Brave | Flatpak |
| Betterbird | Flatpak |
| Signal | Flatpak |
| Steam | Flatpak |
| Google Chrome | Third-party repository |
| Wine | Third-party repository |
| TeamViewer | Stand-alone RPM |

This abstraction keeps the playbooks modular while allowing every application to use the preferred installation mechanism.

---

# ⚙️ Application Configuration

Every application and its installation method is defined in:
```text
vars/applications.yaml
```

The applications to be installed are selected independently in:
```text
group_vars/opensuse.yaml
```

This separates application definitions from host-specific application selection.

Example:
```yaml
applications:

  firefox: true
  chromium: false
  brave: true
  google_chrome: false

  thunderbird: false
  betterbird: true

  steam: true
  wine: true
  teamviewer: false
```

This allows creating different workstation profiles without modifying the playbooks themselves.

---

# 🔄 Repository Updates

Repositories are refreshed automatically during playbook execution.

Where supported, repository GPG keys are imported automatically.

Application-specific repositories are managed independently from the system repositories to keep repository configuration modular.

---

# 🔄 Updating an Existing System

For normal system maintenance:

```bash
sudo zypper ref
sudo zypper up
```

When additional repositories such as Packman are enabled, a distribution upgrade may be required to resolve vendor changes:

```bash
sudo zypper ref
sudo zypper dup --allow-vendor-change
```

This behavior is part of openSUSE's package management and is expected.

---

# 🧪 Verifying Ansible Connectivity
Before running the playbooks, verify that Ansible can connect to the managed host and execute modules successfully.

Verify SSH connectivity:
```bash
ansible all -m ping
```

or, when targeting the opensuse inventory group:
```bash
ansible opensuse -m ping
```

A successful result looks similar to:
```text
suse | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

The pong response confirms that:

- the inventory entry is correct
- SSH connectivity is working
- SSH authentication is working
- Ansible can reach the target host
- the required Python interpreter is available on the target

This only verifies Ansible connectivity. It does not verify that the playbooks or their configuration have been executed successfully.

---

# 📝 Notes

- This project is primarily developed and tested against **openSUSE Leap 16**.
- The playbooks are designed for systems managed through **systemd**, **zypper**, and **SSH**.
- The inventory must be adapted to the target system.
- Application selection is controlled through Ansible variables and does not require modifying the playbooks.
- The playbooks are designed to be idempotent and can be executed repeatedly.
- Some applications require third-party repositories or vendor-provided RPM packages.
- The availability of individual packages may depend on the current openSUSE Leap repositories and third-party repositories.
- The KVM/libvirt environment used during development is a test environment and is not required for using the playbooks on a physical system.

---

# ⚠️ Disclaimer

This project automates changes to system configuration, repositories, installed packages, desktop settings, and other components of an openSUSE system.

Although the playbooks are designed to be safe and idempotent, they should always be reviewed before being executed on a production or otherwise important system.

Third-party repositories and vendor-provided packages are outside the control of this project. Their packages, signing keys, availability, and compatibility may change independently.

**Use this project at your own risk.**

---

# 📜 License

This project is licensed under the MIT License.

See the [`LICENSE`](LICENSE) file for the complete license text.

