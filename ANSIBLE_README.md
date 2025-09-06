# Omarchy Ansible Playbook

This directory contains Ansible playbooks and configuration files to install Omarchy on clean Arch Linux systems.

## Prerequisites

### On the Control Machine (where you run Ansible)
- Ansible 2.9 or later
- Python 3.6 or later
- SSH access to target Arch Linux systems

### On Target Arch Linux Systems
- Clean Arch Linux installation (no disk partitioning/encryption needed)
- Root or sudo access
- Internet connection
- SSH server running (if installing remotely)

## Installation

1. **Install Ansible** (if not already installed):
   ```bash
   # On Arch Linux
   sudo pacman -S ansible
   
   # On Ubuntu/Debian
   sudo apt install ansible
   
   # On macOS
   brew install ansible
   ```

2. **Install required Ansible collections**:
   ```bash
   ansible-galaxy collection install -r ansible-requirements.yml
   ```

3. **Configure inventory**:
   Edit `ansible-inventory.ini` and add your target Arch Linux systems:
   ```ini
   [arch_servers]
   arch-server-1 ansible_host=192.168.1.100 ansible_user=archuser
   arch-server-2 ansible_host=192.168.1.101 ansible_user=archuser
   ```

4. **Test connectivity**:
   ```bash
   ansible all -i ansible-inventory.ini -m ping
   ```

## Usage

### Install Omarchy on all servers in inventory:
```bash
ansible-playbook -i ansible-inventory.ini ansible-playbook.yml
```

### Install on specific server:
```bash
ansible-playbook -i ansible-inventory.ini ansible-playbook.yml --limit arch-server-1
```

### Run with verbose output:
```bash
ansible-playbook -i ansible-inventory.ini ansible-playbook.yml -v
```

### Run with check mode (dry run):
```bash
ansible-playbook -i ansible-inventory.ini ansible-playbook.yml --check
```

## What the Playbook Does

The Ansible playbook replicates the functionality of the original `install.sh` script:

### 1. Package Installation
- Installs all packages from the original `packages.sh` script
- Updates system packages first
- Uses pacman to install packages with `--noconfirm` flag
- Note: Some packages may need to be installed manually from AUR if not available in official repos

### 2. Directory Structure Setup
- Creates the omarchy directory structure in `~/.local/share/omarchy/`
- Sets up proper ownership and permissions

### 3. Configuration Files
- Copies all configuration files from `config/` to `~/.config/`
- Copies default configurations from `default/`
- Sets up theme configurations

### 4. Binary Scripts
- Installs all scripts from `bin/` to `~/.local/share/omarchy/bin/`
- Makes scripts executable
- Adds omarchy bin directory to PATH

### 5. Application Setup
- Copies desktop files for applications
- Sets up web applications (HEY, Basecamp, WhatsApp, etc.)
- Configures GTK themes and icons

### 6. System Services
- Enables and starts required systemd services:
  - Docker
  - UFW firewall
  - Avahi (network discovery)
  - CUPS (printing)
  - IWD (wireless networking)
  - WirePlumber (audio)
  - Power Profiles daemon
  - Plymouth (boot splash)

### 7. Fonts and Themes
- Installs custom Omarchy font
- Sets up theme links and configurations
- Updates font cache

### 8. Final Configuration
- Refreshes various service configurations
- Updates package database
- Shows completion message

## Customization

### Variables
You can customize the installation by modifying variables in the playbook:

```yaml
vars:
  omarchy_user: "{{ ansible_user }}"  # Change target user
  omarchy_home: "/home/{{ omarchy_user }}"  # Change home directory
  omarchy_path: "{{ omarchy_home }}/.local/share/omarchy"  # Change omarchy path
```

### Adding Packages
To add additional packages, modify the `name` list in the "Install base packages" task.

### Adding Web Apps
To add more web applications, add entries to the "Set up web applications" task.

## Troubleshooting

### Common Issues

1. **Permission denied errors**:
   - Ensure the target user has sudo access
   - Check that SSH keys are properly configured

2. **Package installation failures**:
   - Verify internet connectivity on target systems
   - Check that the Arch Linux repositories are properly configured
   - Some packages may be available only in AUR - install them manually with `yay -S package-name`

3. **Service start failures**:
   - Some services may require manual configuration after installation
   - Check systemd logs: `journalctl -u service-name`

4. **Font cache issues**:
   - Run `fc-cache` manually on the target system
   - Ensure font files are properly copied

### Debugging

Run the playbook with increased verbosity:
```bash
ansible-playbook -i ansible-inventory.ini ansible-playbook.yml -vvv
```

Check specific tasks by using tags (if implemented) or running individual tasks.

## Security Notes

- The playbook runs with `become: yes` (root privileges)
- Ensure SSH access is properly secured
- Consider using SSH keys instead of passwords
- Review the playbook before running on production systems

## Differences from Original Script

This Ansible playbook focuses on the core installation without:
- Disk partitioning and encryption setup
- Bootloader configuration
- Chroot operations
- Migration scripts (these are typically run after initial setup)

The playbook is designed for systems that already have a clean Arch Linux installation and just need the Omarchy software and configuration applied.
