# IaC Lab

## Context

This is a hands-on Infrastructure-as-Code lab that allows you to experiment different tools.
For now, I am using **KVM/QEMU/libvirt** to virtualize two machines on a RaspBerry Pi, and configure these using Ansible.

## Setup the lab

### Configure networking

We want our two machines to have a static IP address.
If your machines use the Networking service to manage network interfaces (default initial network interface configuration service on Debian), you can configure them in `/etc/network/interfaces`:
```
# The loopback network interface
auto lo
iface lo inet loopback

# Main network interface
auto enp1s0
iface enp1s0 inet static
    address 192.168.122.100/24
    gateway 192.168.122.1
    dns-nameservers 192.168.122.200 8.8.8.8
```

Same thing for the other machine, but with a different IP address.
**You must change** the IP addresses and the interface name with your own values.

### Add the ansibe user

#### Create user and add in sudo group

On the two machines:
```bash
sudo useradd -m -s /bin/bash ansible
sudo usermod -aG sudo ansible
sudo passwd ansible # Change the password
sudo visudo # Allow 'sudo' group to execute any command
```

#### Configure SSH keys to access your hosts

```bash
ssh-keygen -t ed25519 -C "Some info"
ssh-copy-id -i pubkey.pub ansible@ipaddr1
ssh-copy-id -i pubkey.pub ansible@ipaddr2

# You can test SSH connection
ssh -i privatekey ansible@ipaddr1
ssh -i privatekey ansible@ipaddr2
```

### Put your values into inventory file

The inventory file is `inventories/hosts.ini`: change the values with yours, such as IP addresses, username, and private SSH key file.

## Test the Ansible access to your machines

Now that the setup is done, you can check if Ansible can configure the machines, using this ad-hoc command:
```bash
ansible -i inventories/hosts.ini -m ansible.builtin.ping debians
```

This is not a simple ICMP ping, Ansible tries to push the Python module inside of the machines and execute it.

## Use Ansible playbook

Now that Ansible is ready to push modules into the VMs, you can test the playbook I provided in `playbooks/initial_debian_config.yml`:
```bash
ansible-playbook -i inventories/hosts.ini playbooks/initial_debian_config.yml
```

Launch it a second time to observe the idempotence: nothing changes when you reexecute the same playbook few seconds after the first time.
