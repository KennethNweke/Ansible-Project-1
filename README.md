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

### Setup Used 
![image](https://github.com/user-attachments/assets/e1ce99a0-7690-40f4-9481-58f246aadbed)



