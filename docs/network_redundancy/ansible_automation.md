# Using Ansible to Automate Redundant Firewall Configurations


## Overview

For [this assignment](network_redundancy.md), two closely similar firewalls need to be setup and configured. Rather than typing in commands by hand twice, this task can be automated by leveraging Ansible's VyOS_config module. Initial management involving assigning interface addresses and enabling SSH needs to be done by hand, but further configurations can be easily done in this setup from xubuntu-wan due to its placement in the network with a direct connection to each firewall and the WAN.

## Setting up SSH Key-Based Authentication

Ansible works by issuing commands to specified hosts via SSH. In order to automate this without needing to type in a password on each login, SSH key authentication must be enabled. This involves generating a private-public key combination on the management host, and then copying the manager's public key over SSH to the hosts that are going to be controlled. After authentication on each firewall and a successful copy operation, future logins can be done with a key only.

Generate an SSH key on your management host (in this case xubuntu-wan):

`ssh-keygen`

It's best to secure this key with a password when prompted, but for simplicity it can be left blank.

Now copy this key to each firewall, substituting your key name and remote username/host:

`ssh-copy-id -i ~/.ssh/id_rsa.pub user@host`

Test ssh logins to the two hosts. Key authentication should now work successsfully and you won't be prompted to enter a password.


## Ansible Setup on Management Host

Though Ansible could be run and managed from xubuntu-lan, the function of these firewalls as network devices can complicate setup with temporary firewall rules. Because of this, I setup Ansible on xubuntu-wan so reaching out to the Internet doesn't become an issue during setup. After running my playbook that allowed xubuntu-lan to communicate, I installed Ansible and ran further playbooks on this host. SSH key authentication needs to be configured again if you plan to change management machines.

## Installing Ansible

Ansible installation is simple:

```
sudo apt update -y && sudo apt install -y ansible && sudo apt install -y python3-pip && sudo pip3 install paramiko
```

In my case with VyOS, I found that I needed to install the `paramiko` package for Ansible to run playbooks properly.

## Ansible Configuration

Ansible needs an inventory file to work with remote hosts. Because we're dealing with a small environment, this inventory can be defined directly in `/etc/ansible/hosts`. Append something like this to your file:

```
[firewalls]
10.0.17.13 #vyos1
10.0.17.53 #vyos2

[firewalls:vars]
ansible_network_os=vyos
ansible_user=brandon
```

With this configuration in place, playbooks can be called against the group `firewalls` and commands will be executed against addresses defined here. `[firewalls:vars]` defines a few extra variables that can be changed to best fit the environment. In this case, I have a common user, `vyos`, that will be used to ssh into my firewalls.

## Creating Ansible Playbooks

The playbook that I used to configure my firewalls can be found [here](https://github.com/brandon-wilbur/sec440/blob/master/network_redundancy/firewall-config-playbook.yml). This playbook isn't an entirely conclusive representation of setup on these devices from the start because of initial interface assignments and some other steps I took before deciding to automate, but it contains all firewall rules. Parameters are documented on Ansible's vyos_config module site. Ensure that you configure `save: yes` for each task in your playbook, or else changes won't persist across reboots in VyOS. Entries under `lines:` will be identical to setup commands as if you were in an interactive configuration mode within the VyOS console.

Segment out tasks as much as possible to get more troubleshooting information if issues are raised. Tweaking a file is easier when it's clear that a few commands are wrong with a smaller task. In VyOS, subsequent runs of a playbook aren't likely to misconfigure a system, as configuration directives should overwrite existing ones.

## Running a Playbook

After coming up with a playbook, run it from the management system with the following command:

```
ansible-playbook my_playbook.yml
```

Any errors that are risen should be described at the task level.

## References:
* [https://www.ssh.com/ssh/copy-id](https://www.ssh.com/ssh/copy-id)
* [https://docs.ansible.com/ansible/latest/modules/VyOS_config_module.html](https://docs.ansible.com/ansible/latest/modules/VyOS_config_module.html)
