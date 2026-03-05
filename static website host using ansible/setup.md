# Host Static Website using Nginx

Deploy a static HTML page on your server automatically using Ansible and Nginx.

---

## Project Structure

```
/home/ubuntu/
├── ansible.cfg
├── ansible.html              # your webpage
└── playbooks/
    └── deploy_static_webpage.yml
```

---

## How It Works

```
ansible.html  →  copied to  →  /var/www/html/index.html
                                        ↓
                              Nginx serves it on port 80
                                        ↓
                              http://YOUR_SERVER_IP
```

---

## Run

```bash
cd /home/ubuntu
ansible-playbook playbooks/deploy_static_webpage.yml
```

---

## Verify

Open your browser and visit:

```
http://YOUR_SERVER_IP
```

Or check from terminal:

```bash
ansible webservers -m shell -a "curl -s http://localhost" --become
```
