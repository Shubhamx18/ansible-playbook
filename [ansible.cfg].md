# ansible.cfg — Project Configuration

This file tells Ansible **how to connect** to your servers and **how to behave** by default.
Create it inside your project root — no `-i` flag or extra options needed when running playbooks.

---

## Create the File

```bash
cd ansible-playbook
vim ansible.cfg
```

Paste this content:

```ini
[defaults]
inventory         = ./inventories/dev                    # default inventory
remote_user       = ubuntu                               # SSH login user
private_key_file  = /home/ubuntu/keys/ansible-key.pem   # SSH key path
host_key_checking = False                                # skip fingerprint check
retry_files_enabled = False                              # no .retry files on failure

[privilege_escalation]
become          = True                                   # use sudo by default
become_method   = sudo                                   # sudo method
become_user     = root                                   # become root
become_ask_pass = False                                  # no sudo password prompt

[ssh_connection]
pipelining = True                                        # faster execution
```

---

## What Each Section Does

### [defaults]

| Key | Value | What It Does |
|-----|-------|-------------|
| `inventory` | `./inventories/dev` | Default inventory — no `-i` flag needed for dev |
| `remote_user` | `ubuntu` | SSH user Ansible uses to connect |
| `private_key_file` | `/home/ubuntu/keys/...` | SSH key to authenticate |
| `host_key_checking` | `False` | Skip fingerprint check on first connect |
| `retry_files_enabled` | `False` | Stop creating `.retry` files when playbook fails |

### [privilege_escalation]

| Key | Value | What It Does |
|-----|-------|-------------|
| `become` | `True` | Automatically run tasks as sudo |
| `become_method` | `sudo` | Use sudo to escalate |
| `become_user` | `root` | Escalate to root user |
| `become_ask_pass` | `False` | No password prompt for sudo |

### [ssh_connection]

| Key | Value | What It Does |
|-----|-------|-------------|
| `pipelining` | `True` | Reduces SSH connections — makes playbooks run faster |

---

## How It Changes Your Commands

Without `ansible.cfg` every command needs extra flags:

```bash
# without ansible.cfg — long and repetitive
ansible-playbook -i inventories/dev --user ubuntu --private-key ./keys/ansible-key.pem playbooks/install_nginx.yml
```

With `ansible.cfg` — clean and simple:

```bash
# with ansible.cfg — everything is already set
ansible-playbook playbooks/install_nginx.yml
```

---

## Switch Default Environment

Change the `inventory` line to switch your default environment:

```ini
# use dev by default
inventory = ./inventories/dev

# use staging by default
inventory = ./inventories/staging

# use prd by default  ← not recommended
inventory = ./inventories/prd
```

> Always keep `dev` as default. Pass `-i inventories/prd` explicitly for production.

---

## Where Ansible Looks for ansible.cfg

```
./ansible.cfg              ← your project (highest priority)
      ↓
~/.ansible.cfg             ← your home folder
      ↓
/etc/ansible/ansible.cfg   ← system global (lowest priority)
```

Project-level `ansible.cfg` always wins.
