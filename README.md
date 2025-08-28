# 🚀 Ansible Mini Project - Web Server Automation

A compact Ansible project for automating RHEL-based web server provisioning with application deployment.

---

## 📂 Project Structure
```text
ansible-mini-project/
├── ansible.cfg                # Local Ansible configuration
├── site.yml                   # Master playbook
├── inventory/                 # Dynamic inventory
│   ├── hosts.ini              # Target hosts (INI format)
│   ├── group_vars/            # Group-wide variables
│   │   └── all.yml            # Common variables (e.g., timezone)
│   └── host_vars/             # Host-specific overrides
│       ├── client1.yml        # (e.g., host-specific ports)
│       └── client2.yml
│
├── vault/                     # Encrypted secrets
│   ├── main.yml               # Vault-encrypted variables
│   └── vault.txt              # Vault password (referenced in ansible.cfg)
│
├── playbooks/                 # Component playbooks
│   ├── apache.yml             # Apache installation
│   ├── users.yml              # User management
│   └── repo.yml               # Local repo setup
│
├── roles/
│   ├── apache/                # Apache HTTP Server
│   │   ├── tasks/main.yml     # Installs httpd, configures firewalld
│   │   ├── templates/         # Custom index.html.j2
│   │   └── handlers/main.yml  # Graceful restarts
│   │
│   ├── users/                 # User accounts
│   │   ├── tasks/main.yml     # Creates users/groups
│   │   └── vars/main.yml      # Non-sensitive user data
│   │
│   ├── repo/                  # Offline repo
│   │   ├── tasks/main.yml     # Configures yum repo
│   │   └── files/repo/        # Local RPM storage
│   │       ├── Packages       # Package files
│   │       └── repodata/      # Generated metadata
│   │
│   └── deploy/                # Application deployment
│       ├── tasks/main.yml     # Node.js app deployment
│       ├── templates/         # Nginx proxy configs
│       ├── files/             # Application assets
│       │   ├── apps           # Bundled application
│       │   └── scripts/       # Deployment helpers
│       └── defaults/main.yml  # Version variables
│
└── README.md                  # This documentation
```
📊 Architecture Diagram
https://architecture-diagram.png

---

## 📦 Roles Overview

### 🔹 Apache
- Installs/configures Apache web server  
- Deploys custom `index.html` from template  
- Opens port 80 via firewalld  
- Handles graceful restarts on config changes  

### 🔹 Users
- Creates `admins` and `developers` groups  
- Manages user accounts with:  
  - Proper home directories  
  - Vault-encrypted passwords  
  - SSH key authentication  
- Implements access controls:  
  - Sudo privileges for admins  
  - Passwordless sudo for specific commands  
  - SSH access restrictions  

### 🔹 Repo
- Builds local RPM repository  
- Distributes repo to targets via rsync  
- Configures YUM to:  
  - Prioritize local repo  
  - Disable external repositories  
- Maintains package cache  

⚠ **Important:**  
The `roles/repo/files/repo/Packages` and `roles/repo/files/repo/repodata` directories are **not included** in this repository.  
If you want to use the repo role, you must first create a local YUM repository on your own machine, generate metadata using `createrepo`, and place the `Packages` and `repodata` directories under `roles/repo/files/repo/`.

Example:
```bash
createrepo_c /path/to/roles/repo/files/repo/
```

### 🔹 Deploy
- Installs runtime:  
  - Node.js  
  - npm dependencies  
  - Security fixes via `npm audit fix`  
- Configures services:  
  - Nginx reverse proxy  
  - PM2 process management  
- Ensures:  
  - Auto-start on boot  
  - Zero-downtime reloads  

---

## 🔐 Vault
Sensitive data (passwords, keys) are encrypted using ansible-vault.  
Vault password file is referenced in `ansible.cfg`.

---

## 🚀 Usage

### Run All Roles Together
```bash
ansible-playbook -i inventory/hosts.ini site.yml
```

### Run Specific Role
```bash
ansible-playbook -i inventory/hosts.ini playbooks/<role_name>.yml
```
Example:
```bash
ansible-playbook -i inventory/hosts.ini playbooks/users.yml
ansible-playbook -i inventory/hosts.ini playbooks/repo.yml
ansible-playbook -i inventory/hosts.ini playbooks/deploy.yml
ansible-playbook -i inventory/hosts.ini playbooks/apache.yml
```

### Run with Vault
```bash
ansible-playbook -i inventory/hosts.ini site.yml --ask-vault-pass
```
Or, if `ansible.cfg` already has vault password file:
```bash
ansible-playbook -i inventory/hosts.ini site.yml
```
