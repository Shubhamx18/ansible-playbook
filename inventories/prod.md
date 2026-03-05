# inventories/prd — Production Inventory

This file contains live production server IPs.

> ⚠️ Every command here runs on **real servers**. Always test in `dev` first.

---

## File Content

```ini
[webservers]
prd-server1  ansible_host=20.0.1.10
prd-server2  ansible_host=20.0.1.11

[dbservers]
prd-db1  ansible_host=20.0.2.10

[appservers]
prd-app1  ansible_host=20.0.3.10

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/keys/ansible-key.pem
ansible_python_interpreter=/usr/bin/python3
```

---

## Usage

Production is **never the default**. Always pass `-i inventories/prd` explicitly so you never accidentally hit production.

### Step 1 — Test connectivity first
```bash
ansible -i inventories/prd all -m ping
ansible -i inventories/prd webservers -m ping
```

### Step 2 — Dry run to see what will change
```bash
ansible-playbook -i inventories/prd playbooks/install_nginx.yml --check
```

### Step 3 — Apply only after verifying dry run output
```bash
ansible-playbook -i inventories/prd playbooks/install_nginx.yml
```

### Run on a specific group only
```bash
ansible-playbook -i inventories/prd playbooks/install_nginx.yml --limit webservers
ansible-playbook -i inventories/prd playbooks/install_nginx.yml --limit dbservers
```

---

## Ground Rules

| Rule | Why |
|------|-----|
| Always run `--check` before applying | See what changes before anything happens |
| Never set prd as default in `ansible.cfg` | Prevents accidental runs on live servers |
| Always test in `dev` first | Every change goes `dev` → `staging` → `prd` |
