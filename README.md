# Ansible Playbook — Infrastructure Automation

This project uses Ansible to automate server configuration and application deployment across multiple environments (dev, staging, prd).

---

## 📁 Project Structure

```
ansible-playbook/
├── inventories/
│   ├── dev/
│   │   ├── webservers
│   │   ├── dbservers
│   │   ├── appservers
│   │   └── group_vars/
│   │       ├── all.yml
│   │       ├── webservers.yml
│   │       └── dbservers.yml
│   ├── staging/              # same structure as dev
│   └── prd/                  # same structure as dev
├── roles/
│   ├── common/               # baseline setup for every server
│   ├── nginx/                # nginx install + config
│   └── docker/               # docker install + config
├── host_vars/                # per-server variable overrides
├── keys/                     # SSH keys (never committed)
├── ansible.cfg               # project-level config
├── .gitignore
└── README.md
```

---

## Installation

### 1. Install Ansible

```bash
sudo apt-get update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
ansible --version
```

### 2. Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/ansible-playbook.git
cd ansible-playbook
```

### 3. Add Your SSH Key

```bash
mkdir ~/keys
cd ~/keys
ssh-keygen -t rsa -b 4096 -f ansible-key.pem
chmod 400 ansible-key.pem
```

---

## Inventory Setup

We use `[all:vars]` or `[group:vars]` to define the user, key, and python path once — not repeated on every line.

```ini
[webservers]
server1  ansible_host=10.0.1.10
server2  ansible_host=10.0.1.11

[dbservers]
db1  ansible_host=10.0.2.10

[appservers]
app1  ansible_host=10.0.3.10

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/keys/ansible-key.pem
ansible_python_interpreter=/usr/bin/python3
```

Use `[group:vars]` when different groups need different users or keys:

```ini
[webservers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/keys/ansible-key.pem
ansible_python_interpreter=/usr/bin/python3
```

---

## ansible.cfg

```ini
[defaults]
inventory         = ./inventories/dev
remote_user       = ubuntu
private_key_file  = /home/ubuntu/keys/ansible-key.pem
host_key_checking = False

[privilege_escalation]
become        = True
become_method = sudo
```

Change `inventory` here to switch your default environment without using `-i` every time.

---

## 4 Ways to Set Inventory

| Method | How |
|--------|-----|
| `/etc/ansible/hosts` | Global default — no config needed |
| `-i` flag | `ansible-playbook -i inventories/prd/ ...` |
| `ansible.cfg` | `inventory = ./inventories/dev` |
| Environment variable | `export ANSIBLE_INVENTORY=./inventories/prd` |

**Priority:** `-i` flag → env variable → `ansible.cfg` → `/etc/ansible/hosts`

---

## Test Connectivity

```bash
# Ping all servers
ansible all -m ping

# Ping a specific group
ansible webservers -m ping

# Ping a single server
ansible server1 -m ping
```

---

## Ad-hoc Commands

```bash
# Check memory — all servers
ansible all -m shell -a "free -h" --become

# Check disk space — specific group
ansible webservers -m shell -a "df -h" --become

# Check uptime — single server
ansible server1 -m shell -a "uptime" --become

# Update all servers
ansible all -m shell -a "apt-get update -y" --become

# Update a specific group
ansible webservers -m shell -a "apt-get update -y" --become

# Update a single server
ansible server1 -m shell -a "apt-get update -y" --become
```

---

## Adding a New Group

Create a new file inside the environment folder:

```bash
vim inventories/dev/monitoringservers
```

```ini
[monitoringservers]
monitor1  ansible_host=10.0.4.10
monitor2  ansible_host=10.0.4.11

[monitoringservers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/keys/ansible-key.pem
ansible_python_interpreter=/usr/bin/python3
```

Ansible picks it up automatically — no other changes needed.

```bash
ansible monitoringservers -m ping
ansible-playbook -i inventories/dev/ roles/common/tasks/main.yml --limit monitoringservers
```

---

## Using a Separate Inventory File

For one-off tasks or a completely different setup:

```bash
vim my-custom-inventory
```

```ini
[client_servers]
client1  ansible_host=203.0.113.10
client2  ansible_host=203.0.113.11

[client_servers:vars]
ansible_user=ec2-user
ansible_ssh_private_key_file=/home/ubuntu/keys/client-key.pem
ansible_python_interpreter=/usr/bin/python3
```

Use it directly:

```bash
ansible -i my-custom-inventory all -m ping
ansible-playbook -i my-custom-inventory roles/nginx/tasks/main.yml
```

Or set it as default in `ansible.cfg`:

```ini
[defaults]
inventory = ./my-custom-inventory
```

---

## Running Playbooks

Playbooks live inside the `playbooks/` folder. You always run them like:

```bash
ansible-playbook playbooks/install_nginx.yml
```

```bash
# Default environment (dev)
ansible-playbook playbooks/install_nginx.yml
ansible-playbook playbooks/install_docker.yml
ansible-playbook playbooks/setup_server.yml

# Specific environment using -i flag
ansible-playbook -i inventories/staging/ playbooks/install_nginx.yml
ansible-playbook -i inventories/prd/     playbooks/install_docker.yml

# Specific group only
ansible-playbook -i inventories/prd/ playbooks/install_nginx.yml --limit webservers

# Dry run — always do this before production
ansible-playbook -i inventories/prd/ playbooks/install_nginx.yml --check

# Verbose output — useful for debugging
ansible-playbook playbooks/install_nginx.yml -v
ansible-playbook playbooks/install_nginx.yml -vvv
```

---

## Roles

| Role | What It Does |
|------|-------------|
| `common` | System update, installs curl, vim, git, htop — runs on every server |
| `nginx` | Installs Nginx, deploys config, handles restarts |
| `docker` | Installs Docker CE, adds user to docker group |

```yaml
- name: Setup web server
  hosts: webservers
  become: yes
  roles:
    - common
    - nginx
    - docker
```

---

**Maintainer:** [Shubham Londhe](https://github.com/LondheShubham153)
