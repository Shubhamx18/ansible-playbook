# Playbooks

Playbooks are the files you actually run to configure your servers.

---

## Available Playbooks

| Playbook | Runs On | What It Does |
|----------|---------|-------------|
| `1_server_setup.yml` | webservers | Updates OS, installs basic tools |
| `2_Install_Nginx.yml` | webservers | Installs and starts Nginx |
| `3_Install_docker.yml` | webservers | Installs Docker CE |
| `4_deploy_image_using_docker.yml` | webservers | Pulls image and runs container |

> Run in this order: `1` → `2` → `3` → `4`

---

## How to Run

```bash
# Dev (default)
ansible-playbook playbooks/1_server_setup.yml
ansible-playbook playbooks/2_Install_Nginx.yml
ansible-playbook playbooks/3_Install_docker.yml
ansible-playbook playbooks/4_deploy_image_using_docker.yml

# Staging
ansible-playbook -i inventories/staging playbooks/2_Install_Nginx.yml

# Production — dry run first, then apply
ansible-playbook -i inventories/prd playbooks/2_Install_Nginx.yml --check
ansible-playbook -i inventories/prd playbooks/2_Install_Nginx.yml

# Single server only
ansible-playbook playbooks/2_Install_Nginx.yml --limit server1
```
