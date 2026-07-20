# Ansible Linux Automation

A collection of Ansible playbooks for automating common Linux server administration tasks on Ubuntu/Debian systems.

This project demonstrates practical infrastructure automation including web server deployment, database management, Node.js application deployment, storage management, and network information collection.

---

## Features

- Apache Web Server Installation & Configuration
- MySQL Installation & Secure Configuration
- MySQL Database Restore
- MySQL Data Directory Relocation
- PHP & MySQL Connection Page
- Node.js + PM2 Deployment
- LVM Creation & Extension
- Disk Partitioning & Mounting
- Network Information Collection
- Modular Role-Based Ansible Structure
- Centralized Inventory & Configuration
- Ansible Vault Support

---

## Repository Structure

```text
ansible/
├── ansible.cfg
├── inventory/
│   ├── hosts
│   ├── group_vars/
│   └── host_vars/
│
├── vault/
│   └── secrets.yml
│
├── roles/
│   ├── common/
│   ├── apache/
│   ├── mysql/
│   ├── mysql_restore/
│   ├── mysql_relocate/
│   ├── php_mysql/
│   ├── nodejs/
│   ├── lvm/
│   ├── lvm_extend/
│   ├── disk/
│   └── network_info/
│
├── files/
│   └── dump.sql
│
├── site.yml
├── web.yml
├── db.yml
├── db_restore.yml
├── db_relocate.yml
├── db_connect.yml
├── lvm.yml
├── extend_lvm.yml
├── disk.yml
└── network_info.yml
```

---

## Requirements

| Software  | Version                    |
| --------- | -------------------------- |
| Ansible   | 2.14+                      |
| Python    | 3.8+                       |
| Target OS | Ubuntu 20.04+ / Debian 11+ |

### Required Collections

```bash
ansible-galaxy collection install community.mysql
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix
```

---

## Quick Start

### Clone the repository

```bash
git clone https://github.com/BoraHamarat001/ansible.git
cd ansible
```

### Configure Inventory

Edit the inventory file with your own servers.

```ini
[webservers]
192.168.1.XX

[dbservers]
192.168.1.XX
```

### Configure Vault

Edit your secrets.

```bash
nano vault/secrets.yml
```

Encrypt the file.

```bash
ansible-vault encrypt vault/secrets.yml
```

Edit later if needed.

```bash
ansible-vault edit vault/secrets.yml
```

### Configure SSH

```bash
ssh-keygen -t ed25519 -f ~/.ssh/ansible -C "ansible"

ssh-copy-id -i ~/.ssh/ansible.pub user@server
```

---

# Playbook Examples

## Apache Web Server

```bash
ansible-playbook web.yml
```

Update only the configuration.

```bash
ansible-playbook web.yml --tags config
```

Run against a specific host.

```bash
ansible-playbook web.yml --limit 192.168.1.45
```

---

## MySQL

Install and configure MySQL.

```bash
ansible-playbook db.yml --ask-vault-pass
```

Run only security tasks.

```bash
ansible-playbook db.yml --ask-vault-pass --tags security
```

Restore a database.

```bash
ansible-playbook db_restore.yml --ask-vault-pass
```

Relocate MySQL data directory.

```bash
ansible-playbook db_relocate.yml --ask-vault-pass
```

---

## PHP + MySQL

```bash
ansible-playbook db_connect.yml --ask-vault-pass
```

---

## Node.js Application

```bash
ansible-playbook site.yml --tags nodejs --ask-vault-pass
```

---

## Storage Management

Disk partitioning.

```bash
ansible-playbook disk.yml
```

Create an LVM.

```bash
ansible-playbook lvm.yml
```

Extend an existing LVM.

```bash
ansible-playbook extend_lvm.yml
```

Specify a custom disk.

```bash
ansible-playbook lvm.yml \
-e "lvm_disk_device=/dev/sdc lvm_vg_name=data_vg"
```

---

## Network Information

Collect server information and publish it through Apache.

```bash
ansible-playbook network_info.yml
```

---

## Deploy Everything

```bash
ansible-playbook site.yml --ask-vault-pass
```

---

## Available Tags

| Tag      | Description           |
| -------- | --------------------- |
| always   | Always executed       |
| apache   | Apache tasks          |
| mysql    | MySQL tasks           |
| nodejs   | Node.js deployment    |
| pm2      | PM2 management        |
| lvm      | LVM tasks             |
| disk     | Disk partitioning     |
| network  | Network information   |
| packages | Package installation  |
| config   | Configuration tasks   |
| security | Security tasks        |
| restore  | Database restore      |
| relocate | MySQL data relocation |

---

## Variable Priority

Variables are loaded in the following order.

```text
roles/<role>/defaults/main.yml

inventory/group_vars/all.yml

inventory/group_vars/<group>.yml

inventory/host_vars/<host>.yml

Command line (-e key=value)
```

---

## Security Notes

- Never commit unencrypted secrets.
- Store Vault passwords securely.
- Add sensitive files such as `.vault_pass` to `.gitignore`.
- Consider using environment variables or CI/CD secret management for Vault passwords.

---

## Notes

- All playbooks are designed to be idempotent.
- Service restarts are handled using Ansible handlers.
- The repository follows a role-based project structure for better scalability and maintainability.
