# setup_linux_servers_ansible_vmware

Setup Linux servers via Ansible. VMware Workstation Pro is the hypervisor I am using for this demonstration. This is tested to work on Windows.

## List of Virtual Machines

Ubuntu Desktop
- Name: Ansible Controller
- Network Adapter: NAT
    - IP Address: 192.168.182.128/24
- Memory: 4096 MB
- Processors: 2
- Hard Disk Size: 30 GB
- OS: Ubuntu Desktop

Ubuntu Server 1:
- Name: Web Server
- Network Adapter: NAT
    - IP Address: 192.168.182.137/24
- Memory: 3100 MB
- Processors: 1
- Hard Disk Size: 25 GB
- OS: Ubuntu Server

Ubuntu Server 2:
- Name: DB Server
- Network Adapter: NAT
    - IP Address: 192.168.182.134/24
- Memory: 3100 MB
- Processors: 1
- Hard Disk Size: 25 GB
- OS: Ubuntu Server

Debian Server 1:
- Name: File Server
- Network Adapter: NAT
    - IP Address: 192.168.182.136/24
- Memory: 2048 MB
- Processors: 1
- Hard Disk Size: 35 GB
- OS: Debian

## Virtual Machine Setup

You will create four virtual machines with VMware. The first VM is for Ubuntu Desktop. This VM will be used as the Ansible controller to run playbooks on the other VMs.
The second and third VMs are for Ubuntu Server. The last VM is for Debian.

## Installation 

Install the ansible package on Ubuntu Desktop
```
sudo apt install ansible
```

Install openssh server on Ubuntu servers
```
sudo apt install openssh-server
```

Install openssh server on Debian servers
```
sudo dnf install openssh-server
```

## Setup SSH Keys

### Linux Servers Key

Create ssh key for the linux servers
```
ssh-keygen -t ed25519 -C "salmaan default"
```

The file path for the private key should be:
```
/home/salmaan/.ssh/linux_servers_ansible
```

Copy the linux servers ssh key to the linux servers. Replace the IP address to the IP address of the linux servers.
```
ssh-copy-id -i ~/.ssh/linux_servers_ansible.pub sage@192.168.182.137
```

Check if performing ssh to the linux servers occurs automatically without a password confirmation
```
ssh sage@192.168.182.137
```

### Ansible Key

Create the ssh key for ansible
```
ssh-keygen -t ed25519 -C "ansible"
```

The file path for the private key should be:
```
/home/salmaan/.ssh/ansible
```

Copy the ansible ssh key to the linux servers. Replace the IP address to the IP address of the linux servers.
```
ssh-copy-id -i ~/.ssh/ansible.pub sage@192.168.182.137
```

Check if performing ssh to the linux servers with the ansible ssh key occurs automatically without a password confirmation
```
ssh -i ~/.ssh/ansible 192.168.182.137
```

## Setup and configure Ansible

Ping all of the hosts
```
ansible all -m ping
```

List all of the hosts
```
ansible all --list-hosts
```

Gather facts about the target systems
```
ansible all -m gather_facts
```

Gather facts about a particular target system
```
ansible all -m gather_facts --limit 192.168.182.137
```

## Running Elevated Commands with Ansible

Make ansible use sudo with --ask-become-pass
```
ansible all -m apt -a update_cache=true --become --ask-become-pass
```

Install the vim package via the apt module
```
ansible all -m apt -a name=vim --become --ask-become-pass
```

Install the snapd package and make sure it's the latest version available
```
ansible all -m apt -a "name=snapd state=latest" --become --ask-become-pass
```

Upgrade all the package updates that are available via apt
```
ansible all -m apt -a upgrade=dist --become --ask-become-pass
```

## Writing and Running an Ansible Playbook

Run the bootstrap playbook
```
ansible-playbook --ask-become-pass bootstrap.yml
```

Run the site playbook
```
ansible-playbook --become --ask-become-pass site.yml
```

## Linux Terminal Tips and Tricks

Get the information of the Operating System on Linux
```
cat /etc/os-release
```

List all users
```
compgen -u
```

List last 20 users
```
tail -n 20 /etc/passwd
```

## The When Conditional

Get the ansible_distribution information for a Linux server
```
ansible all -m gather_facts --limit 192.168.182.137 | grep ansible_distribution
```

## Ansible Tags

List available tags in a playbook
```
ansible-playbook --list-tags site.yml
```

Running a playbook while targeting specific tags
```
ansible-playbook --tags db --ask-become-pass site.yml
ansible-playbook --tags debian --ask-become-pass site.yml
ansible-playbook --tags apache --ask-become-pass site.yml
```

Running a playbook while specifying multiple tags
```
ansible-playbook --tags "apache,db" --ask-become-pass site.yml
```

## Ubuntu Server 26.04 Sudo Bug for Ansible
Ubuntu server uses sudo-rs instead of sudo.

Uninstall sudo-rs and install sudo:
```
sudo apt remove sudo-rs
sudo apt install sudo
```

## Resources
- [Ansible community documentation - Ansible](https://docs.ansible.com/)
- [VirtualBox can't operate in VMX mode - superuser](https://superuser.com/questions/1845776/virtualbox-cant-operate-in-vmx-mode)
