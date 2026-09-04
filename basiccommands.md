# Ansible Basics

## 1. Core Concepts

### Inventory

An **inventory** is a collection of servers/hosts where Ansible performs tasks.

Default inventory:

```text
/etc/ansible/hosts
```

### Module

`-m` specifies the Ansible module.

Example:

```bash
ansible localhost -m ping
```

### Ad-Hoc Commands

An **ad-hoc command** performs a task directly from the command line without a playbook.

Example:

```bash
ansible localhost -a "git --version"
```

### Playbook

A **playbook** contains Ansible tasks written in YAML and is used to automate multiple tasks.

---

# 2. Test Localhost

```bash
ansible localhost -m ping
```

Expected output:

```text
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

# 3. Common Ad-Hoc Commands

### Install Git

```bash
ansible localhost -a "sudo apt install git -y"
```

### Check Git Version

```bash
ansible localhost -a "git --version"
```

### Install Maven

```bash
ansible localhost -a "sudo apt install maven -y"
```

### Check Maven

```bash
ansible localhost -a "mvn --version"
```

### Create a File

```bash
ansible localhost -a "touch filename.txt"
```

### Install Apache

```bash
ansible localhost -a "sudo apt install apache2 -y"
```

### Start Apache

```bash
ansible localhost -a "sudo systemctl start apache2"
```

### Check Apache

```bash
ansible localhost -a "systemctl status apache2"
```

### Create User

```bash
ansible localhost -a "sudo useradd user1"
```

### Check User

```bash
ansible localhost -a "cat /etc/passwd | grep user1"
```

### Remove Packages

```bash
ansible localhost -a "sudo apt remove git maven apache2 -y"
```

---

# 4. Inventory Setup

Create the inventory directory/file:

```bash
sudo mkdir -p /etc/ansible
sudo vi /etc/ansible/hosts
```

Example:

```ini
[slaves]
slave1 ansible_host=172.31.3.37
slave2 ansible_host=172.31.3.38

[slaves:vars]
ansible_user=root
ansible_ssh_private_key_file=/root/.ssh/id_ed25519
```

You can also group servers:

```ini
[prod]
172.31.3.37
172.31.3.38

[dev]
172.31.3.39
172.31.3.40
```

---

# 5. SSH Key Setup

On the **master/control node**, generate an SSH key:

```bash
ssh-keygen
```

Copy the public key to each managed server:

```bash
ssh-copy-id root@<server-ip>
```

Test the connection:

```bash
ssh root@<server-ip>
```

---

# 6. Root SSH Login

By default, root SSH login may be disabled.

On the managed machines:

```bash
sudo vi /etc/ssh/sshd_config
```

Configure:

```text
PermitRootLogin yes
PasswordAuthentication yes
```

Then restart SSH:

```bash
sudo systemctl restart ssh
```

Check status:

```bash
sudo systemctl status ssh
```

> **Security note:** Enabling root/password SSH login is generally not recommended for production. SSH keys with a non-root Ansible user and `--become` are safer.

---

# 7. Test All Servers

Ping all hosts:

```bash
ansible all -m ping
```

View inventory:

```bash
ansible-inventory --list
```

Update all servers:

```bash
ansible all -a "sudo apt update -y"
```

Install Git:

```bash
ansible all -a "sudo apt install git -y"
```

Check Git:

```bash
ansible all -a "git --version"
```

Install Maven and Apache:

```bash
ansible all -a "sudo apt install maven apache2 -y"
```

Start Apache:

```bash
ansible all -a "sudo systemctl start apache2"
```

Create user:

```bash
ansible all -a "sudo useradd user1"
```

Check user:

```bash
ansible all -a "cat /etc/passwd | grep user1"
```

---

# 8. Run Commands on Specific Groups

If the inventory contains:

```ini
[prod]
172.31.3.37
172.31.3.38
```

Run a command only on `prod` servers:

```bash
ansible prod -a 'echo "hello world"'
```

Run on all servers:

```bash
ansible all -a "hostname"
```

### Important

```text
ansible all
       ↓
All inventory hosts

ansible prod
       ↓
Only hosts in [prod]

ansible localhost
       ↓
Only localhost
```

---

# Quick Reference

| Command                          | Purpose                          |
| -------------------------------- | -------------------------------- |
| `ansible all -m ping`            | Test all hosts                   |
| `ansible-inventory --list`       | Display inventory                |
| `ansible all -a "command"`       | Run command on all hosts         |
| `ansible prod -a "command"`      | Run command only on `prod`       |
| `ansible localhost -a "command"` | Run command on localhost         |
| `-m`                             | Specify module                   |
| `-a`                             | Specify module/command arguments |
| `--become`                       | Execute with elevated privileges |
| `ssh-keygen`                     | Generate SSH keys                |
| `ssh-copy-id`                    | Copy SSH public key to server    |
