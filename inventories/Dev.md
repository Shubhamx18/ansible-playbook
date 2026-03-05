# inventories/dev — Development Inventory

This file contains **development server IPs only**.
Safe to test and experiment here — no risk to live servers.

> Location: `inventories/dev` inside your project

---

## File Content

```ini
[webservers]
dev-server1  ansible_host=10.0.1.10
dev-server2  ansible_host=10.0.1.11

[dbservers]
dev-db1  ansible_host=10.0.2.10

[appservers]
dev-app1  ansible_host=10.0.3.10

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/keys/ansible-key.pem
ansible_python_interpreter=/usr/bin/python3
```

---

## Set as Default in ansible.cfg

```ini
[defaults]
inventory = ./inventories/dev
```

---

## Usage

Since dev is the default, no `-i` flag needed:

```bash
# Test connectivity
ansible all -m ping
ansible webservers -m ping

# Run a playbook
ansible-playbook playbooks/install_nginx.yml
ansible-playbook playbooks/install_docker.yml

# Target a specific group only
ansible-playbook playbooks/install_nginx.yml --limit webservers

# Dry run first
ansible-playbook playbooks/install_nginx.yml --check
```

---

