# Ubuntu Server Setup Playbook

This playbook provides comprehensive initial setup and security hardening for Ubuntu servers, particularly useful for cloud VMs.

## Features

### Security Hardening
- **SSH Configuration**: Disables password authentication, root login, and hardens SSH settings
- **Firewall**: Configures UFW with sensible defaults
- **Fail2ban**: Protects against brute force attacks
- **System Hardening**: Applies kernel security parameters and disables unused protocols
- **Security Tools**: Installs and configures rkhunter, chkrootkit, and other monitoring tools

### System Configuration
- **Package Updates**: Performs full system update and upgrade
- **Essential Packages**: Installs commonly needed tools (curl, wget, vim, htop, etc.)
- **Timezone**: Configures server timezone
- **Automatic Updates**: Sets up unattended security updates

### Monitoring & Maintenance
- **Log Monitoring**: Configures logwatch for system monitoring
- **Rootkit Detection**: Sets up rkhunter and chkrootkit
- **System Maintenance**: Configures automatic cleanup of old packages

## Prerequisites

1. **Ansible Collections**: Install required collections:
   ```bash
   ansible-galaxy install -r requirements.yml
   ```

2. **SSH Key Access**: Ensure you have SSH key-based access to target servers
3. **Sudo Access**: The user must have sudo privileges on target servers

## Usage

### 1. Setup Inventory
Copy the example inventory and customize for your servers:
```bash
cp inventory.example.yml inventory.yml
# Edit inventory.yml with your server details
```

### 2. Run the Playbook
```bash
# Run against all servers in inventory
ansible-playbook -i inventory.yml playbooks/ubuntu-server-setup-playbook.yml

# Run against specific group
ansible-playbook -i inventory.yml playbooks/ubuntu-server-setup-playbook.yml --limit ubuntu_servers

# Run against specific server
ansible-playbook -i inventory.yml playbooks/ubuntu-server-setup-playbook.yml --limit server1
```

### 3. Verify Setup
After running the playbook, verify the setup:
```bash
# Check SSH configuration
ansible -i inventory.yml ubuntu_servers -m command -a "sudo sshd -t"

# Check firewall status
ansible -i inventory.yml ubuntu_servers -m command -a "sudo ufw status"

# Check fail2ban status
ansible -i inventory.yml ubuntu_servers -m command -a "sudo fail2ban-client status"
```

## Customization

### Variables
You can customize the setup by modifying variables in your inventory file or creating a separate vars file:

```yaml
# SSH Configuration
ssh_port: 22                          # SSH port (default: 22)
ssh_permit_root_login: "no"           # Allow root login
ssh_password_authentication: "no"     # Allow password auth

# Security
firewall_allowed_ports:               # Additional ports to open
  - "22/tcp"
  - "80/tcp"
  - "443/tcp"

# System
server_timezone: "UTC"                # Server timezone
auto_upgrades_enabled: true           # Enable automatic updates
auto_reboot_enabled: false            # Allow automatic reboots
auto_reboot_time: "06:00"            # Reboot time if enabled
```

### Adding Custom Ports
To open additional firewall ports, add them to the `firewall_allowed_ports` list in your inventory:

```yaml
firewall_allowed_ports:
  - "22/tcp"      # SSH
  - "80/tcp"      # HTTP
  - "443/tcp"     # HTTPS
  - "8080/tcp"    # Custom application
  - "3306/tcp"    # MySQL (if needed)
```

## Important Notes

⚠️ **Before Running**: Ensure you have SSH key access to all target servers, as the playbook will disable password authentication.

⚠️ **SSH Port**: If you change the SSH port, make sure to update your SSH client configuration and firewall rules accordingly.

⚠️ **First Run**: The playbook may require the server to reboot. Plan accordingly for production systems.

## Post-Setup Recommendations

1. **Test SSH Access**: Verify you can still connect after the security changes
2. **Monitor Logs**: Check `/var/log/fail2ban.log` and `/var/log/auth.log` for security events
3. **Review Firewall**: Use `sudo ufw status numbered` to review firewall rules
4. **Update Regularly**: Run the `apt-upgrade-playbook.yml` regularly to keep systems updated

## Troubleshooting

### SSH Connection Issues
If you get locked out after running the playbook:
1. Check if you're using the correct SSH port
2. Ensure your SSH key is properly configured
3. Verify the user account has the correct public key in `~/.ssh/authorized_keys`

### Firewall Issues
If services are not accessible:
1. Check UFW status: `sudo ufw status numbered`
2. Add required ports: `sudo ufw allow PORT/tcp`
3. Verify service is running: `sudo systemctl status SERVICE`

### Access Cloud Console
For cloud VMs, you can usually access the console through your cloud provider's web interface if SSH access is lost.