# Ansible Playbook: Ansible-k3s

## Description
There are two ansible playbooks in here:

- One to deploy a k3s cluster
- One to teardown a k3s cluster

## Requirements

- Python3 and pip installed on your machine

## Instructions

Instructions can be found [here](https://homelab-coral.vercel.app/iac/ansible/#run-k3s-playbook) on how to run this playbook.

## Linting and syntax-check

You can run yamllint to lint your yaml via

```shell
yamllint .
```

You can run syntax-check to verify the playbook's imports via
```shell
ansible-playbook playbook/playbook.yml --syntax-check
```

## Configuration

- **Inventory File**: `inventory/hosts.ini`
  - Update this file to define your target hosts and also contains your token, so make sure you encrypt this file with vault.
Example hosts.ini file below.
```ini
[servers]
192.168.1.1
192.168.1.2
192.168.1.3	

# variables for all the servers
[servers:vars]
ansible_ssh_user=myusername
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_port=34168

# secret, make sure this file is in ansible vault to encrypt it
token=mytoken 
```
- **Variables**: `inventory/group_vars.yml`
  - Customize variables in this file to configure the playbook behavior. There are also variables inside for each role in `roles/{rolename}/vars/main.yml` and `roles/{rolename}/defaults/main.yml`
- **Playbooks**: `playbook/*.yml`
  - `provision.yml` is the playbook to run to provision/create the k3s cluster.
  - `destroy.yml` is the playbook to run to destroy/tear down the k3s cluster.
- **roles**: `roles/{rolename}`
  - prereq- role to run before creating k3s cluster. It syncs up the timezone in all nodes, enables ipv4 forwarding, allows api and etcd ports to go through firewall.
  - k3s_server- role to spin up our k3s cluster. It installs the binary, starts up each node, and setups up kubectl in each node.
## Misc

- This playbook was heavily influenced by [k3 ansible's playbook](https://github.com/k3s-io/k3s-ansible/tree/master)
