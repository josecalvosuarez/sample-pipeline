# Environment Setup Guide

This guide provides step-by-step instructions for setting up a development environment with four virtual machines using Vagrant and VirtualBox. Each VM has a specific role: Jenkins, Git repository, Configuration Management, and Minikube.

The Vagrantfile automatically detects your host CPU architecture and selects the correct VM images — it works on macOS (Intel and Apple Silicon) and Windows (x86_64).

---

## Prerequisites

### macOS

#### 1. Install Homebrew
If Homebrew is not installed, run:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 2. Install VirtualBox
```bash
brew install --cask virtualbox
```

#### 3. Install Vagrant
```bash
brew install vagrant
```

**NOTE**: The plugin manager is broken in Vagrant 2.4.2. If you hit issues, manually install 2.4.3 from https://developer.hashicorp.com/vagrant/install

#### 4. Install the Vagrant Host Manager plugin
```bash
vagrant plugin install vagrant-hostmanager
```

---

### Windows

#### 1. Install Git for Windows
Download and install from https://git-scm.com/download/win

This also installs Git Bash, which provides an SSH client used by `vagrant ssh`.

#### 2. Install VirtualBox
Download and install from https://www.virtualbox.org/wiki/Downloads

#### 3. Install Vagrant
Download and install from https://developer.hashicorp.com/vagrant/install

#### 4. Install the Vagrant Host Manager plugin

Open **Command Prompt or PowerShell as Administrator** and run:
```powershell
vagrant plugin install vagrant-hostmanager
```

> **Important**: The Host Manager plugin modifies `C:\Windows\System32\drivers\etc\hosts` to add VM hostnames. This requires Administrator privileges. You must run all `vagrant up` and `vagrant reload` commands from an **Administrator** terminal.

#### 5. Enable Virtualization in BIOS
Ensure **Intel VT-x** or **AMD-V** is enabled in your BIOS/UEFI settings. Most modern PCs have this enabled by default.

#### 6. Disable Hyper-V (if enabled)
VirtualBox conflicts with Hyper-V on Windows. If Hyper-V is active, run the following in an **Administrator** PowerShell and reboot:
```powershell
bcdedit /set hypervisorlaunchtype off
```

---

### Verify Installations (all platforms)

```bash
vagrant --version
virtualbox --help
```

Ensure no errors are reported before proceeding.

---

## Step-by-Step Environment Deployment

1. **Clone the Repository**
   ```bash
   git clone <repo-url>
   cd <repo-name>
   ```

2. **Initialize the Vagrant Environment**

   > **Windows users**: Run this from an **Administrator** Command Prompt or PowerShell.

   ```bash
   vagrant up
   ```

   This will:
   - Download the necessary Vagrant boxes (if not already downloaded)
   - Create and provision all four VMs
   - Add hostname entries to your hosts file (`/etc/hosts` on Mac/Linux, `C:\Windows\System32\drivers\etc\hosts` on Windows)

3. **Access the Virtual Machines**

   SSH into a specific VM:
   ```bash
   vagrant ssh jenkins
   vagrant ssh git
   vagrant ssh config-mgmt
   vagrant ssh minikube
   ```

   Check the status of all VMs:
   ```bash
   vagrant global-status
   ```

4. **Verify Network Communication**

   From the Jenkins VM, ping another VM by IP or hostname:
   ```bash
   ping 192.168.56.102    # Git VM by IP
   ping git.local         # Git VM by hostname
   ```

   Available hostnames:
   - `jenkins.local` — 192.168.56.101
   - `git.local` — 192.168.56.102
   - `config-mgmt.local` — 192.168.56.103
   - `minikube.local` — 192.168.56.104

5. **Access Jenkins**

   Open a browser and navigate to:
   ```
   http://jenkins.local:8080
   ```
   Retrieve the initial admin password from inside the Jenkins VM:
   ```bash
   vagrant ssh jenkins
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

---

## Troubleshooting

### VM Deployment Issues

**macOS — VirtualBox permission errors**
Go to **System Settings → Privacy & Security** and approve the VirtualBox kernel extension if prompted.

**Windows — VirtualBox fails to start VMs**
- Ensure Hyper-V is disabled (see Prerequisites above).
- Ensure virtualization is enabled in BIOS.
- Check that no other hypervisor (WSL2, Docker Desktop with Hyper-V backend) is active.

**Windows — hostmanager cannot update hosts file**
Run your terminal as Administrator. The plugin needs write access to `C:\Windows\System32\drivers\etc\hosts`.

### Network Issues
- Ensure the Host-Only Network adapter (`192.168.56.0/24`) is configured in VirtualBox under **File → Host Network Manager**.
- Run `vagrant reload` to restart VMs and reapply network settings.

### Vagrant Issues
- Run `vagrant reload` to restart the VMs.
- Run `vagrant provision` to reapply the provisioning scripts without destroying the VMs.
- Run `vagrant destroy -f && vagrant up` to start from scratch.

---

This concludes the setup process. If you encounter issues, consult your instructor or refer to the Vagrant and VirtualBox documentation. Happy learning!
