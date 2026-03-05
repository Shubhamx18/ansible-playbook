# Ansible Modules — Complete Reference

Modules are the building blocks of every Ansible task.
Each task calls one module to do one specific job on the server.

```yaml
- name: Install nginx
  apt:                  ← this is a module
    name: nginx
    state: present
```

---

## 1. Package Management

### apt — Install / Remove Packages

```yaml
- name: Install a package
  apt:
    name: nginx
    state: present        # present = install, absent = remove, latest = upgrade

- name: Install multiple packages
  apt:
    name:
      - curl
      - vim
      - git
    state: present

- name: Update package cache
  apt:
    update_cache: yes

- name: Update cache and install
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Remove a package
  apt:
    name: nginx
    state: absent
```

---

## 2. Service Management

### service — Start / Stop / Restart services

```yaml
- name: Start a service
  service:
    name: nginx
    state: started

- name: Stop a service
  service:
    name: nginx
    state: stopped

- name: Restart a service
  service:
    name: nginx
    state: restarted

- name: Enable service on boot
  service:
    name: nginx
    state: started
    enabled: yes

- name: Reload service
  service:
    name: nginx
    state: reloaded
```

### systemd — Manage systemd services

```yaml
- name: Start a service
  systemd:
    name: nginx
    state: started

- name: Stop a service
  systemd:
    name: nginx
    state: stopped

- name: Restart a service
  systemd:
    name: nginx
    state: restarted

- name: Enable service on boot
  systemd:
    name: nginx
    enabled: yes

- name: Reload systemd daemon
  systemd:
    daemon_reload: yes

- name: Start and enable on boot
  systemd:
    name: nginx
    state: started
    enabled: yes
    daemon_reload: yes
```

> Use `systemd` instead of `service` when working with Ubuntu — it gives you more control like `daemon_reload`.

---

## 3. File Management

### file — Create / Delete files and folders

```yaml
- name: Create a directory
  file:
    path: /etc/myapp
    state: directory
    mode: '0755'

- name: Create an empty file
  file:
    path: /etc/myapp/config.txt
    state: touch

- name: Delete a file
  file:
    path: /etc/myapp/config.txt
    state: absent

- name: Set file permissions
  file:
    path: /home/ubuntu/keys/ansible-key.pem
    mode: '0400'
    owner: ubuntu
    group: ubuntu
```

### copy — Copy files to server

```yaml
- name: Copy a file to server
  copy:
    src: ./files/nginx.conf
    dest: /etc/nginx/nginx.conf
    owner: root
    mode: '0644'

- name: Write content directly to a file
  copy:
    dest: /etc/myapp/config.yml
    content: |
      host: localhost
      port: 3000
```

### template — Copy Jinja2 template with variables

```yaml
- name: Deploy config from template
  template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    mode: '0644'
```

### fetch — Download file from server to local

```yaml
- name: Fetch log file from server
  fetch:
    src: /var/log/nginx/error.log
    dest: ./logs/
    flat: yes
```

---

## 4. Command Execution

### command — Run a command (no shell features)

```yaml
- name: Run a simple command
  command: uptime

- name: Run command and save output
  command: docker --version
  register: docker_version

- name: Print output
  debug:
    var: docker_version.stdout
```

### shell — Run a command (with shell features like pipes)

```yaml
- name: Run shell command with pipe
  shell: cat /etc/passwd | grep ubuntu

- name: Run multiple commands
  shell: |
    cd /home/ubuntu
    mkdir -p myapp
    echo "done"

- name: Use environment variables
  shell: echo $HOME
```

> Use `command` when you don't need pipes or shell features. Use `shell` when you do.

---

## 5. User Management

### user — Create / Delete users

```yaml
- name: Create a user
  user:
    name: deploy
    state: present
    shell: /bin/bash
    home: /home/deploy

- name: Add user to a group
  user:
    name: ubuntu
    groups: docker
    append: yes

- name: Delete a user
  user:
    name: deploy
    state: absent
    remove: yes
```

---

## 6. System

### ping — Test connectivity

```yaml
- name: Test server connectivity
  ping:
```

### debug — Print a message or variable

```yaml
- name: Print a message
  debug:
    msg: "Server setup complete"

- name: Print a variable
  debug:
    var: ansible_hostname

- name: Print registered output
  debug:
    var: my_variable.stdout
```

### set_fact — Set a variable during playbook run

```yaml
- name: Set a custom variable
  set_fact:
    app_port: 3000
    app_env: production
```

### timezone — Set server timezone

```yaml
- name: Set timezone
  timezone:
    name: UTC
```

---

## 7. Network & Firewall

### ufw — Manage firewall rules (Ubuntu)

```yaml
- name: Allow SSH
  ufw:
    rule: allow
    port: 22
    proto: tcp

- name: Allow HTTP
  ufw:
    rule: allow
    port: 80
    proto: tcp

- name: Allow HTTPS
  ufw:
    rule: allow
    port: 443
    proto: tcp

- name: Enable firewall
  ufw:
    state: enabled

- name: Deny a port
  ufw:
    rule: deny
    port: 8080
```

---

## 8. Git

### git — Clone a repository

```yaml
- name: Clone a repo
  git:
    repo: https://github.com/user/myapp.git
    dest: /home/ubuntu/myapp
    version: main

- name: Pull latest changes
  git:
    repo: https://github.com/user/myapp.git
    dest: /home/ubuntu/myapp
    update: yes
```

---

## 9. Docker

### docker_container — Run / Stop containers

```yaml
- name: Run a container
  docker_container:
    name: myapp
    image: nginx:latest
    state: started
    ports:
      - "80:80"
    restart_policy: always

- name: Stop a container
  docker_container:
    name: myapp
    state: stopped

- name: Remove a container
  docker_container:
    name: myapp
    state: absent
```

### docker_image — Pull Docker images

```yaml
- name: Pull an image
  docker_image:
    name: nginx
    tag: latest
    source: pull
```

---

## 10. Handlers

Handlers are special tasks that only run when **notified** by another task — usually to restart a service after a config change.

```yaml
# In tasks
- name: Copy Nginx config
  copy:
    src: nginx.conf
    dest: /etc/nginx/nginx.conf
  notify: Restart Nginx          ← triggers the handler

# In handlers
handlers:
  - name: Restart Nginx
    service:
      name: nginx
      state: restarted
```

> Handler runs only once at the end of the play, even if notified multiple times.

---

## Quick Reference

| Module | Use For |
|--------|---------|
| `apt` | Install / remove packages |
| `service` | Start / stop / restart services |
| `systemd` | Manage systemd services + daemon reload |
| `file` | Create / delete files and folders |
| `copy` | Copy files to server |
| `template` | Deploy Jinja2 config templates |
| `fetch` | Download files from server |
| `command` | Run simple commands |
| `shell` | Run commands with pipes / shell features |
| `user` | Create / delete users |
| `ping` | Test connectivity |
| `debug` | Print variables and messages |
| `ufw` | Manage firewall rules |
| `git` | Clone / pull repositories |
| `docker_container` | Run / stop containers |
| `docker_image` | Pull Docker images |
