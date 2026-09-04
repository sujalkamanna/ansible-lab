# Ansible Modules & Ad-Hoc Commands

## 1. Introduction

**Ansible modules** are reusable units of code that Ansible uses to perform specific tasks on managed/remote machines.

For example, Ansible can use modules to:

* Install or uninstall packages
* Start, stop, or restart services
* Create users
* Copy files from the control node to managed nodes
* Execute commands
* Manage system resources

---

# 2. Ansible Ad-Hoc Commands

An **ad-hoc command** allows us to execute a task directly from the command line without creating a playbook.

General syntax:

```bash
ansible <pattern> -m <module> -a "<arguments>"
```

Where:

* `ansible` → Ansible command
* `<pattern>` → Hosts on which the command should run, e.g. `all`
* `-m` → Specifies the Ansible module
* `-a` → Specifies the arguments passed to the module

---

# 3. Executing System Commands

We can execute a normal Linux command on remote machines using Ansible.

Example:

```bash
ansible all -a "yum install -y git"
```

Here:

* `all` → Execute on all managed nodes
* `-a` → Pass the command as an argument
* `yum install -y git` → Linux command executed on the remote machine

This is an **ad-hoc command using the default `command` module**.

> **Note:** For package installation, it is generally better to use Ansible's package-management modules instead of directly running `yum` or `apt` commands.

---

# 4. Using Ansible Modules

Ansible provides many built-in modules for different tasks.

Example:

```bash
ansible all -m apt -a "name=git state=present"
```

Here:

* `-m apt` → Uses the `apt` module
* `name=git` → Specifies the package name
* `state=present` → Ensures that the package is installed

---

# 5. Package Management Modules

Common package-management modules include:

* `apt` → Debian/Ubuntu-based systems
* `yum` → RHEL/CentOS-based systems
* `dnf` → Modern RHEL/Fedora-based systems
* `package` → Generic package manager module

## Common Package States

| State     | Meaning                                                           |
| --------- | ----------------------------------------------------------------- |
| `present` | Install the package if it is not installed                        |
| `absent`  | Uninstall/remove the package                                      |
| `latest`  | Install the package if needed and update it to the latest version |

### Install Git

```bash
ansible all -m apt -a "name=git state=present"
```

### Remove Git

```bash
ansible all -m apt -a "name=git state=absent"
```

### Install Maven

```bash
ansible all -m apt -a "name=maven state=present"
```

### Install Apache

```bash
ansible all -m apt -a "name=apache2 state=present"
```

---

# 6. Checking Installed Software

We can use ad-hoc commands to check whether software is installed and determine its version.

For example:

```bash
ansible all -a "git --version"
```

For Apache:

```bash
ansible all -a "apache2 -v"
```

> The `command` module is used by default when no module is explicitly specified.

---

# 7. Service Module

The Ansible `service` module is used to manage services on remote machines.

General syntax:

```bash
ansible all -m service -a "name=<service_name> state=<state>"
```

## Common Service States

| State       | Meaning             |
| ----------- | ------------------- |
| `started`   | Start the service   |
| `stopped`   | Stop the service    |
| `restarted` | Restart the service |

### Start Apache

```bash
ansible all -m service -a "name=apache2 state=started"
```

### Stop Apache

```bash
ansible all -m service -a "name=apache2 state=stopped"
```

### Restart Apache

```bash
ansible all -m service -a "name=apache2 state=restarted"
```

### Check Apache Status

```bash
ansible all -a "systemctl status apache2"
```

---

# 8. Enable a Service at Boot

The `service` module can also be used with `enabled` to ensure that a service starts automatically when the system boots.

Example:

```bash
ansible all -m service -a "name=apache2 state=started enabled=yes"
```

Here:

* `state=started` → Start Apache now
* `enabled=yes` → Enable Apache to start automatically at boot

> `enabled` is a parameter, not a service `state`.

---

# 9. User Module

The Ansible `user` module is used to create, modify, or remove users.

### Create a User

```bash
ansible all -m user -a "name=user1 state=present"
```

This ensures that `user1` exists on all managed nodes.

### Check Users

We can view the `/etc/passwd` file using:

```bash
ansible all -a "cat /etc/passwd"
```

---

# 10. Creating Files

We can execute Linux commands remotely to create files.

Example:

```bash
ansible all -a "touch file1.txt"
```

This creates `file1.txt` on all managed machines.

For file management, Ansible also provides the `file` module.

For example:

```bash
ansible all -m file -a "path=/home/ubuntu/file1.txt state=touch"
```

Using the Ansible module is generally preferable when we want Ansible to manage the desired state of a file.

---

# 11. Copy Module

The Ansible `copy` module is used to copy files from the **Ansible control node (master)** to managed nodes.

### Step 1: Create a File on the Control Node

On the Ansible control node:

```bash
touch python1.py
```

### Step 2: Copy the File to Managed Nodes

```bash
ansible all -m copy -a "src=python1.py dest=/home/ubuntu/"
```

This copies:

```text
Control Node
    |
    |--- python1.py
    |
    +------> Managed Node 1
    |
    +------> Managed Node 2
    |
    +------> Managed Node 3
```

The file will be placed at:

```text
/home/ubuntu/python1.py
```

on the managed nodes.

### Step 3: Verify the File

```bash
ansible all -a "ls /home/ubuntu/"
```

---

# 12. Using `--become`

Some operations require **root privileges**.

Ansible provides the `--become` option to execute tasks with elevated privileges.

Example:

```bash
ansible all -m apt -a "name=git state=absent" --become
```

Here:

```text
--become
   ↓
Elevated/root privileges
```

This is commonly used for tasks such as:

* Installing packages
* Removing packages
* Managing system services
* Modifying system configuration
* Creating system users

Depending on the configuration, Ansible may use `sudo` to obtain the required privileges.

---

# 13. Important Commands Summary

## Package Management

### Install Git

```bash
ansible all -m apt -a "name=git state=present"
```

### Remove Git

```bash
ansible all -m apt -a "name=git state=absent"
```

### Install Maven

```bash
ansible all -m apt -a "name=maven state=present"
```

### Install Apache

```bash
ansible all -m apt -a "name=apache2 state=present"
```

---

## Service Management

### Start Apache

```bash
ansible all -m service -a "name=apache2 state=started"
```

### Stop Apache

```bash
ansible all -m service -a "name=apache2 state=stopped"
```

### Restart Apache

```bash
ansible all -m service -a "name=apache2 state=restarted"
```

### Check Apache Status

```bash
ansible all -a "systemctl status apache2"
```

---

## User Management

### Create User

```bash
ansible all -m user -a "name=user1 state=present"
```

### Check Users

```bash
ansible all -a "cat /etc/passwd"
```

---

## File Management

### Create a File

```bash
ansible all -a "touch file1.txt"
```

### Copy a File

On the control node:

```bash
touch python1.py
```

Then:

```bash
ansible all -m copy -a "src=python1.py dest=/home/ubuntu/"
```

Verify:

```bash
ansible all -a "ls /home/ubuntu/"
```

---

# 14. Quick Reference: Modules

| Module    | Purpose                                       |
| --------- | --------------------------------------------- |
| `command` | Execute commands on remote machines           |
| `shell`   | Execute commands through a shell              |
| `apt`     | Manage packages on Debian/Ubuntu              |
| `yum`     | Manage packages on older RHEL/CentOS systems  |
| `dnf`     | Manage packages on modern RHEL/Fedora systems |
| `service` | Manage services                               |
| `systemd` | Manage systemd services                       |
| `user`    | Manage users                                  |
| `file`    | Manage files and directories                  |
| `copy`    | Copy files from control node to managed nodes |

---

# 15. Key Concepts to Remember

### `-m` = Module

Example:

```bash
-m apt
```

means use the `apt` module.

### `-a` = Arguments

Example:

```bash
-a "name=git state=present"
```

passes arguments to the module.

### `state`

The meaning of `state` depends on the module.

For package modules:

```text
present → installed
absent  → removed
latest  → latest version
```

For service management:

```text
started   → service is running
stopped   → service is stopped
restarted → service is restarted
```

### `--become`

Used when elevated/root privileges are required:

```bash
ansible all -m apt -a "name=git state=present" --become
```

---

# 16. Ad-Hoc Command vs Module-Based Command

### Ad-Hoc Linux Command

```bash
ansible all -a "yum install -y git"
```

Ansible executes the Linux command directly.

### Module-Based Command

```bash
ansible all -m apt -a "name=git state=present"
```

Ansible uses the `apt` module to achieve the desired package state.