# Ansible

## What is Ansible?

**Ansible** is an open-source **IT automation and configuration management tool** used to automate tasks across multiple servers and systems.

It allows you to manage many machines from a single system called the **Control Node**.

Instead of manually connecting to every server and running commands, you can define what needs to be done once and let Ansible execute it across your infrastructure.

### In simple words

> **Ansible = Automate tasks across multiple machines using simple, human-readable configuration.**

---

# What Does Ansible Do?

Ansible can automate tasks such as:

* Installing and removing software
* Starting, stopping, and restarting services
* Creating and managing users
* Copying files
* Managing configuration files
* Deploying applications
* Updating servers
* Managing web servers such as Apache and Nginx
* Managing databases
* Provisioning infrastructure
* Performing server configuration
* Application deployment
* Continuous deployment

For example, instead of manually running:

```bash
sudo apt install git -y
```

on 10 servers, Ansible can do it on all of them:

```bash
ansible all -m apt -a "name=git state=present" --become
```

---

# Why Use Ansible?

Without automation:

```text
Admin
  │
  ├── SSH → Server 1 → Install Git
  ├── SSH → Server 2 → Install Git
  ├── SSH → Server 3 → Install Git
  ├── SSH → Server 4 → Install Git
  └── SSH → Server 5 → Install Git
```

With Ansible:

```text
                 Ansible
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
     Server 1    Server 2    Server 3
        │           │           │
       Git         Git         Git
```

You define the task once, and Ansible executes it across the required hosts.

---

# Ansible Architecture

Ansible follows a **Control Node → Managed Nodes** architecture.

```text
                    ┌──────────────────────┐
                    │    Control Node      │
                    │                      │
                    │      Ansible         │
                    │      Inventory       │
                    │      Playbooks       │
                    │      Modules         │
                    │      SSH Keys         │
                    └──────────┬───────────┘
                               │
                         SSH / WinRM
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
       ┌────────────┐   ┌────────────┐   ┌────────────┐
       │ Managed    │   │ Managed    │   │ Managed    │
       │ Node 1     │   │ Node 2     │   │ Node 3     │
       │            │   │            │   │            │
       │ Apache     │   │ Nginx      │   │ Docker     │
       └────────────┘   └────────────┘   └────────────┘
```

## 1. Control Node

The **Control Node** is the machine where Ansible is installed and from which automation is executed.

It contains:

* Ansible installation
* Inventory
* Playbooks
* Roles
* Configuration
* SSH keys

Example:

```text
Master / Control Node
        ↓
Runs Ansible
        ↓
Manages remote servers
```

---

## 2. Managed Nodes

**Managed Nodes** are the servers or machines that Ansible manages.

They can be:

* Linux servers
* Windows servers
* Cloud instances
* Virtual machines
* Containers
* Network devices

Ansible generally does not require an Ansible agent to be installed on Linux managed nodes.

---

## 3. Inventory

The **inventory** tells Ansible which machines it should manage.

Common inventory location:

```text
/etc/ansible/hosts
```

Example:

```ini
[prod]
192.168.1.10
192.168.1.11

[dev]
192.168.1.20
192.168.1.21
```

Now we can target specific groups:

```bash
ansible prod -m ping
```

or all hosts:

```bash
ansible all -m ping
```

---

# Ansible Modules

**Modules** are the building blocks Ansible uses to perform tasks.

Examples:

| Module    | Purpose                       |
| --------- | ----------------------------- |
| `apt`     | Manage Debian/Ubuntu packages |
| `yum`     | Manage packages               |
| `service` | Manage services               |
| `user`    | Manage users                  |
| `copy`    | Copy files                    |
| `file`    | Manage files/directories      |
| `command` | Execute commands              |
| `shell`   | Execute shell commands        |
| `ping`    | Test connectivity             |

Example:

```bash
ansible all -m apt -a "name=git state=present"
```

Here:

```text
-m apt
   ↓
Use the apt module

-a
   ↓
Pass module arguments

name=git
   ↓
Package name

state=present
   ↓
Ensure package is installed
```

---

# Ad-Hoc Commands

**Ad-hoc commands** are one-time commands executed directly from the terminal.

Example:

```bash
ansible all -m ping
```

```bash
ansible all -a "hostname"
```

```bash
ansible all -a "git --version"
```

They are useful for:

* Quick testing
* Troubleshooting
* Checking servers
* Performing simple one-time tasks

---

# Playbooks

A **playbook** is a YAML file containing a set of Ansible tasks.

Instead of executing commands individually:

```bash
ansible all -a "apt update"
ansible all -a "apt install git -y"
ansible all -a "systemctl start apache2"
```

we can define the automation in a playbook.

Example:

```yaml
---
- name: Install and start Apache
  hosts: web
  become: true

  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
        update_cache: true

    - name: Start Apache
      service:
        name: apache2
        state: started
```

Run it with:

```bash
ansible-playbook apache.yml
```

---

# Basic Ansible Structure

A typical Ansible project can look like this:

```text
ansible-project/
│
├── inventory/
│   └── hosts
│
├── playbooks/
│   ├── install-apache.yml
│   ├── install-nginx.yml
│   └── create-user.yml
│
├── roles/
│   └── webserver/
│
├── files/
│
├── templates/
│
└── ansible.cfg
```

### Main Components

```text
Inventory
    ↓
Defines WHERE

Playbook
    ↓
Defines WHAT

Modules
    ↓
Define HOW

Tasks
    ↓
Individual actions

Roles
    ↓
Organize reusable automation
```

---

# How Ansible Works

A simplified workflow:

```text
        User
         │
         ▼
   Ansible Command
         │
         ▼
     Inventory
         │
         ▼
      Playbook
         │
         ▼
      Modules
         │
         ▼
     SSH / WinRM
         │
         ▼
   Managed Nodes
         │
         ▼
     Task Result
```

For Linux systems, Ansible commonly connects using **SSH**.

Example:

```bash
ansible all -m ping
```

Ansible:

1. Reads the inventory
2. Finds the target hosts
3. Connects to them
4. Executes the required module/task
5. Collects the result
6. Reports success, failure, or changes

---

# Idempotency

One of the important features of Ansible is **idempotency**.

It means that running the same task multiple times should not continuously change the system once the desired state has already been reached.

For example:

```yaml
- name: Install Git
  apt:
    name: git
    state: present
```

If Git is already installed, Ansible doesn't need to reinstall it.

The desired state is:

```text
Git → Installed
```

Ansible checks the current state and makes changes only when necessary.

---

# Ansible vs Manual Administration

### Manual

```text
Login to Server 1
      ↓
Run commands

Login to Server 2
      ↓
Run commands

Login to Server 3
      ↓
Run commands
```

### Ansible

```text
Write automation once
        ↓
      Ansible
        ↓
 ┌──────┼──────┐
 ▼      ▼      ▼
 S1     S2     S3
```

This makes infrastructure management **faster, repeatable, consistent, and easier to maintain**.

---

# Key Terms

| Term               | Meaning                                            |
| ------------------ | -------------------------------------------------- |
| **Control Node**   | Machine where Ansible runs                         |
| **Managed Node**   | Server managed by Ansible                          |
| **Inventory**      | List/group of managed hosts                        |
| **Module**         | Performs a specific operation                      |
| **Task**           | A single unit of work                              |
| **Play**           | Maps tasks to a group of hosts                     |
| **Playbook**       | YAML file containing plays/tasks                   |
| **Role**           | Reusable structure for Ansible automation          |
| **Ad-Hoc Command** | One-time command executed from CLI                 |
| **`--become`**     | Executes tasks with elevated privileges            |
| **SSH**            | Common connection method for Linux hosts           |
| **Idempotency**    | Repeated execution produces the same desired state |

---

# In One Picture

```text
                         ANSIBLE
                            │
              ┌─────────────┴─────────────┐
              │                           │
          Inventory                   Playbooks
              │                           │
       "WHERE to run"              "WHAT to do"
              │                           │
              └─────────────┬─────────────┘
                            │
                         Modules
                            │
                       "HOW to do it"
                            │
                            ▼
                    SSH / WinRM
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
         Server 1        Server 2        Server 3
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                       Desired State
```