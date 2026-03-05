# /etc/ansible/hosts — Default System Inventory

This is the **default inventory file** that Ansible looks for automatically.
No `-i` flag needed. No `ansible.cfg` needed. Just run Ansible and it picks this up.

> Location is always: `/etc/ansible/hosts`

---

## Edit the File

```bash
sudo vim /etc/ansible/hosts
```

---

## File Content

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

---

## Usage

Since this is the system default, no flag is needed:

```bash
# Test all servers
ansible all -m ping

# Test a specific group
ansible webservers -m ping

# Run a playbook
ansible-playbook playbooks/install_nginx.yml
```

---

## When to Use

- Quick one-off tasks on your local machine
- Learning and testing Ansible for the first time
- When you only have one environment and don't need separate dev/prd files

> For real projects with multiple environments, use separate `dev` and `prd` inventory files with `-i` flag instead.
