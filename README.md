# Ansible Playbook: Ansible-k3s


> *Important Note:* Currently this playbook does not work without disable firewall (ufw).

## Description
There are two ansible playbooks in here:

- One to deploy a k3s cluster
- One to teardown a k3s cluster

## Requirements

- Python3 and pip installed on your machine
- netaddr package installed via `pip install netaddr`

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
[master]
192.168.1.1
192.168.1.2
192.168.1.3

[k3s_cluster:children]
master

# variables for all the servers
[k3s_cluster:vars]
ansible_ssh_user=kazzam
ansible_user=kazzam
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_port=34168

# secret, make sure this file is in ansible vault to encrypt it
k3s_token=f88f42c779ce08c518d67be49e0781f2
```
- **Variables**: `inventory/group_vars.yml`
  - Customize variables in this file to configure the playbook behavior. There may also be variables inside for each role in `roles/{rolename}/vars/main.yml` and `roles/{rolename}/defaults/main.yml`
- **Playbooks**: `playbook/*.yml`
  - `provision.yml` is the playbook to run to provision/create the k3s cluster.
  - `destroy.yml` is the playbook to run to destroy/tear down the k3s cluster.
- **roles**: `roles/{rolename}`
  - prereq- Role to run before creating k3s cluster. It syncs up the timezone in all nodes, enables ipv4 forwarding, allows api and etcd ports to go through firewall.
  - download- Another role to run before creating k3s cluster. It downloads the binary files for k3s
  - k3s_server- Spin our k3s cluster. It installs the binary, starts up each node, and setups up kubectl in each node. It sets up MetalLB as the service load balancer and kube-vip as the control plane load balancer.
  - k3s_server-post- Deploy MetalLB pool of ips set in inventory all.yml
  - reset- Destroy cluster and all associated files

## Extra config

### Allow kubectl from local machine
```shell
mkdir -p ~/.kube
scp kazzam@192.168.1.6:~/.kube/config ~/.kube/config
```

### Validating install

https://technotim.live/posts/k3s-etcd-ansible/#testing-your-cluster

## Misc
- This playbook was heavily influenced by [k3 ansible's playbook](https://github.com/k3s-io/k3s-ansible/tree/master) for the first iteration and is almost an exact replica of [technotim's ansible's playbook](https://github.com/techno-tim/k3s-ansible)