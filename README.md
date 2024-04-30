# Ansible Playbook: Ansible-k3s


> *Important Note:* Currently this playbook does not work without disable firewall (ufw).

## Description
There are a few ansible playbooks in here:

Instructions can be found [here](https://homelab-coral.vercel.app/iac/ansible/#run-k3s-playbook) on how to run this playbook, or below

- Provision a new K3s cluster
  
```bash
ansible-vault encrypt inventory/hosts.ini
ansible-playbook playbook/provision.yml -i inventory/hosts.ini -K --ask-vault-pass
``` 

- teardown a K3s cluster
```bash
ansible-vault encrypt inventory/hosts.ini
ansible-playbook playbook/destroy.yml -i inventory/hosts.ini -K --ask-vault-pass
```

- Install Traefik and its dependencies like cert-manager
```bash
ansible-vault encrypt inventory/group_vars/traefik.yml
ansible-playbook -i inventory/local.ini playbook/traefik.yml --ask-vault-pass
```

## Requirements

- Python3 and pip installed on your machine
- netaddr package installed via `pip install netaddr`

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

- **Inventory File**: 
  - `inventory/hosts.ini`
  Update this file to define your target hosts and also contains your token, so make sure you encrypt this file with vault.
Example hosts.ini file below.
```ini
[master]
192.168.1.1
192.168.1.2
192.168.1.3

[node]
192.168.1.4

[k3s_cluster:children]
master
node

# variables for all the servers
[k3s_cluster:vars]
ansible_ssh_user=kazzam
ansible_user=kazzam
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_port=34168

# secret, make sure this file is in ansible vault to encrypt it
k3s_token=f88f42c779ce08c518d67be49e0781f2
```
- `inventory/local.ini`
  Used for installing traefik
```ini
[traefik]
localhost
```
- **Variables**: `inventory/group_vars.yml`
  - `k3s_cluster.yml` = Customize variables in this file to configure the behavior for k3s playbook. There may also be variables inside for each role in `roles/{rolename}/vars/main.yml` and `roles/{rolename}/defaults/main.yml`
  - `traefik.yml` - This file is encrypted, used for Traefik playbook, example config below.
  ```yaml
  ---
  # needed to tell ansible we are not ssh'ing and just running locally
  ansible_connection: local
  
  # Traefik variables
  username: "karl"
  password: "password"
  ingress_subdomain: "local.example.com"
  helm_version: "27.0.2"
  
  # cert-manager variables
  email: "example@gmail.com"
  domain: "example.com"
  cloudflare_token: "api_token"
  
  # cert common variables between staging and production
  common_name: "*.local.example.com"
  dns_name_1: "local.example.com"
  dns_name_2: "*.local.example.com"
  
  # staging
  metadata_name_staging: "local-example-com-staging"
  staging_secret_name: "local-example-com-staging-tls"
  staging_issuer_ref: "letsencrypt-staging"
  
  # production
  metadata_name_production: "local-example-com-production"
  production_secret_name: "local-example-com-tls"
  production_issuer_ref: "letsencrypt-production"
  # value should be true only second time running playbook
  # (once confirming that staging works)
  create_production_cert: "false"
  ```
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

### Validating k3s install

https://technotim.live/posts/k3s-etcd-ansible/#testing-your-cluster

Files are under `/validation folder`

## Misc
- This playbook was heavily influenced by [k3 ansible's playbook](https://github.com/k3s-io/k3s-ansible/tree/master) for the first iteration and is almost an exact replica of [technotim's ansible's playbook](https://github.com/techno-tim/k3s-ansible)