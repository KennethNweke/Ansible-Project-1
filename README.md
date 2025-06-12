# Ansible-Project-1
Ansible was utilized to automate the configuration backup process for multiple Cisco network devices. A designated inventory group was created to manage a router and two switches, allowing their running configurations to be systematically retrieved and stored using a timestamped naming convention for both folders and files.

## Objective
The objective of this project was to demonstrate effective network automation by leveraging Ansible to manage and interact with Cisco IOS devices. The target network included a Cisco 1900 router and two switches, configured with the IP addresses 192.168.125.1, 192.168.125.7, and 192.168.125.8. A custom Ansible playbook was developed to connect to each device, retrieve its hostname and full running configuration, and save the output locally within a timestamped directory structure. This solution ensures reliable, consistent backups and contributes to scalable network configuration management and disaster recovery preparedness.

## Key Requirements
- SSH Enabled on Devices: All Cisco routers and switches must have SSH access configured to allow Ansible to connect and execute commands remotely.
- IP Address Configuration: Each network device is assigned a static IP address to ensure consistent connectivity (e.g., 192.168.125.1, 192.168.125.7, 192.168.125.8).
- Physical Connectivity: Devices must be physically connected to the network and powered on to respond to SSH requests during backup execution.
- Network Reachability: The Ansible control node must be able to reach each device over the network (verified using ping or ssh).
- Ansible Control Node: A dedicated host machine (Ubuntu-based) with Ansible installed is required to execute the playbook and manage devices.
- VirtualBox Bridged Adapter: The Ansible control node runs inside a VirtualBox virtual machine configured with a bridged network adapter, enabling it to operate on the same LAN as the physical Cisco devices.

## STopology Used 
![image](https://github.com/user-attachments/assets/e1ce99a0-7690-40f4-9481-58f246aadbed)

### 📁 Ansible Inventory (`inventory.ini`)

```ini
# Define the group of Cisco network devices
[cisco_devices]
ROUTER-1 ansible_host=192.168.125.1  # Cisco 1900 router
SWITCH-1 ansible_host=192.168.125.2  # Cisco switch 1
SWITCH-2 ansible_host=192.168.125.3  # Cisco switch 2

# Group variables applied to all devices in [cisco_devices]
[cisco_devices:vars]
ansible_user=cisco                      # SSH username
ansible_password=cisco                  # SSH password
ansible_port=22                         # Default SSH port
ansible_become=yes                      # Enable privilege escalation
ansible_become_method=enable            # Use 'enable' mode for Cisco devices
ansible_become_password=cisco           # Password for enable mode
ansible_network_os=cisco.ios.ios        # Specify Cisco IOS network OS plugin
ansible_connection=network_cli          # Use CLI connection for network devices
```

### 📄 Ansible Playbook: `backup_playbook.yml`

```yaml
---
- name: Backup Cisco Device Configurations
  hosts: cisco_devices          # Target group of Cisco devices defined in the inventory
  gather_facts: no              # Skip fact gathering (not needed for network devices)
  vars:
    backup_base_dir: "{{ playbook_dir }}/backups"  # Base directory for storing backups

  pre_tasks:
    - name: Generate timestamp (run once on localhost)
      delegate_to: localhost
      run_once: true
      set_fact:
        global_timestamp: "{{ lookup('pipe', 'date +%Y-%m-%d_%H-%M-%S') }}"  # Get current date/time

    - name: Set shared backup directory path
      set_fact:
        backup_dir: "{{ backup_base_dir }}/{{ global_timestamp }}"  # Full backup directory path with timestamp

  tasks:
    - name: Create local backup directory (once for all backups)
      delegate_to: localhost
      run_once: true
      file:
        path: "{{ backup_dir }}"   # Create the backup directory locally
        state: directory
        mode: '0755'

    - name: Get device hostname
      ios_command:
        commands: "show running-config | include ^hostname"  # Fetch the hostname from running config
      register: hostname_output

    - name: Extract hostname
      set_fact:
        device_hostname: "{{ hostname_output.stdout[0].split(' ')[1] }}"  # Parse hostname from command output

    - name: Backup running config
      ios_command:
        commands: "show running-config"  # Retrieve full running configuration
      register: running_config

    - name: Ensure backup directory exists on localhost
      delegate_to: localhost
      file:
        path: "{{ backup_dir }}"  # Double-check directory exists before saving files
        state: directory
        mode: '0755'

    - name: Save config to file
      delegate_to: localhost
      copy:
        content: "{{ running_config.stdout[0] }}"  # Save config content to file
        dest: "{{ backup_dir }}/{{ device_hostname }}_{{ global_timestamp }}.cfg"  # Filename includes hostname and timestamp
```
