# Ansible Automation for AWS Multi-Server Fleet

> Centralized configuration management and web application deployment across multiple AWS EC2 instances using Ansible.

🟧 `AWS EC2` &nbsp;&nbsp; ⬛ `ANSIBLE` &nbsp;&nbsp; 🟧 `UBUNTU` &nbsp;&nbsp; 🟩 `NGINX` &nbsp;&nbsp; 🟦 `SSH` &nbsp;&nbsp; 🟪 `YAML`

---

## Project Overview

This project demonstrates how **Ansible can centrally manage and configure multiple AWS EC2 servers**.

Instead of manually logging in to each server to install packages, configure services, update configuration files, and deploy applications, the required configuration is defined once and applied consistently across the server fleet.

The environment consists of:

- 1 Ubuntu EC2 Ansible Control Node
- 3 Ubuntu EC2 Managed Nodes
- Nginx web server
- Static web application
- SSH-based passwordless authentication
- Ansible Playbooks, Roles, Variables, Templates, and Handlers

---

## Problem Statement

Managing multiple servers manually becomes difficult as infrastructure grows.

Manual administration can result in:

- Configuration drift between servers
- Inconsistent package or service configuration
- Missed configuration changes
- Repetitive administrative work
- Human error

For example, a configuration change may be applied correctly on one server but missed on another.

This project solves that problem by using **Ansible configuration management** to maintain a consistent desired state across multiple AWS EC2 servers.

---

## Solution

A dedicated Ubuntu EC2 instance acts as the **Ansible Control Node** and manages three Ubuntu EC2 web servers through SSH.

Ansible automates:

- Nginx installation
- Service configuration
- Nginx configuration deployment
- Static application deployment
- Consistent configuration across the server fleet
- Configuration-triggered Nginx reloads
- Reusable configuration through Roles and Variables

---

## Technology Stack

### Cloud
**AWS EC2**

Hosts the Ansible Control Node and managed servers.

### Operating System
**Ubuntu Linux**

Used across both the controller and managed nodes.

### Configuration Management
**Ansible**

Automates server configuration and application deployment.

### Web Server
**Nginx**

Serves the deployed static web application.

### Connectivity
**SSH**

Provides secure communication between the Ansible Control Node and managed nodes.

### Configuration
**YAML · Jinja2**

Used for Playbooks, Roles, Variables, Tasks, Handlers, and Nginx templating.

### Version Control
**Git · GitHub**

Used to version and showcase the automation project.

---

## Project Structure

```text
ansible-aws-multiserver/
│
├── README.md
├── .gitignore
├── ansible.cfg
├── inventory.ini
├── playbook.yml
│
└── roles/
    └── webserver/
        ├── defaults/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        ├── tasks/
        │   └── main.yml
        ├── files/
        │   └── index.html
        └── templates/
            └── nginx.conf.j2
```

---

# Implementation

## 1. AWS Environment

Four Ubuntu EC2 instances are used:

```text
ansible-control
Web_Server_01
Web_Server_02
Web_Server_03
```

`ansible-control` acts as the centralized Ansible Control Node.

The remaining three instances form the managed web-server fleet.

---

## 2. Ansible Control Node

Ansible is installed only on the Control Node.

```bash
sudo apt update
sudo apt install ansible -y
```

Verify the installation:

```bash
ansible --version
```

Python availability can be verified using:

```bash
python3 --version
```

---

## 3. Passwordless SSH Authentication

An SSH key pair is generated on the Ansible Control Node:

```bash
ssh-keygen -t rsa
```

The Control Node public key is then authorized on each managed EC2 instance.

Example:

```bash
ssh-copy-id -f "-o IdentityFile=/home/ubuntu/aws-learning.pem" ubuntu@<WEB_SERVER_01_IP>
```

Repeat for the remaining managed nodes:

```bash
ssh-copy-id -f "-o IdentityFile=/home/ubuntu/aws-learning.pem" ubuntu@<WEB_SERVER_02_IP>
```

```bash
ssh-copy-id -f "-o IdentityFile=/home/ubuntu/aws-learning.pem" ubuntu@<WEB_SERVER_03_IP>
```

After the public key is authorized, the Control Node can connect to the managed nodes without repeatedly specifying the AWS PEM key:

```bash
ssh ubuntu@<MANAGED_NODE_IP>
```

Private SSH keys and AWS PEM files are **not stored in this repository**.

---

## 4. Ansible Inventory and Configuration

The three managed EC2 instances are grouped under:

```text
webservers
```

The project-level `ansible.cfg` defines the inventory and remote Ubuntu user.

Actual infrastructure addresses can be replaced with placeholders before publishing the repository.

---

## 5. Verify Ansible Connectivity

Before applying configuration, connectivity to the complete server fleet is verified using:

```bash
ansible webservers -m ping
```

The hostnames of all managed servers can also be verified centrally:

```bash
ansible webservers -m command -a "hostname"
```

Successful responses confirm that the Control Node can communicate with the managed nodes through SSH.

---

## 6. Create the Webserver Role

The web-server configuration is organized using an Ansible Role:

```bash
ansible-galaxy role init roles/webserver
```

The Role contains the tasks, variables, static application, Nginx template, and handler required to configure the managed web-server fleet.

This keeps the main Playbook small and makes the automation easier to maintain and reuse.

---

## 7. Configuration Automation

The `webserver` Role automates the required state of the managed servers:

```text
Update package information
        ↓
Install Nginx
        ↓
Start and enable Nginx
        ↓
Deploy Nginx configuration
        ↓
Deploy web application
        ↓
Maintain consistent configuration
```

Reusable values are maintained through Ansible Variables, while the Nginx configuration is maintained through a Jinja2 Template.

When the managed Nginx configuration changes, the corresponding task notifies a Handler to reload Nginx.

---

## 8. Application Deployment

The static web application is maintained inside:

```text
roles/webserver/files/index.html
```

Ansible deploys the application consistently across all managed web servers under the configured Nginx web root.

Application availability can be validated using:

```bash
curl http://<MANAGED_NODE_IP>
```

or through the corresponding web-server endpoint in a browser.

---

## Idempotency

The project also demonstrates **Ansible idempotency**.

Running the same Playbook repeatedly maintains the desired configuration rather than unnecessarily applying the same changes again.

If the managed nodes already match the desired state, the corresponding idempotent Tasks do not need to modify them again.

---

## Core Ansible Concepts Used

`Inventory` · `Ad-Hoc Commands` · `Playbooks` · `Roles` · `Variables` · `Jinja2 Templates` · `Handlers` · `Privilege Escalation` · `Idempotency`

---

# Useful Commands

### Install Ansible

```bash
sudo apt update
sudo apt install ansible -y
```

### Verify Ansible

```bash
ansible --version
```

### Generate Control Node SSH key

```bash
ssh-keygen -t rsa
```

### Configure passwordless SSH

```bash
ssh-copy-id -f "-o IdentityFile=/home/ubuntu/aws-learning.pem" ubuntu@<MANAGED_NODE_IP>
```

### Verify passwordless SSH

```bash
ssh ubuntu@<MANAGED_NODE_IP>
```

### Test all managed nodes

```bash
ansible webservers -m ping
```

### Check hostnames across the fleet

```bash
ansible webservers -m command -a "hostname"
```

### Create the Webserver Role

```bash
ansible-galaxy role init roles/webserver
```

### Validate Playbook syntax

```bash
ansible-playbook --syntax-check playbook.yml
```

### Execute the Playbook

```bash
ansible-playbook playbook.yml
```

### Verify Nginx is running

```bash
ansible webservers -b -m shell -a "systemctl is-active nginx"
```

Expected:

```text
active
```

### Verify Nginx starts after reboot

```bash
ansible webservers -b -m shell -a "systemctl is-enabled nginx"
```

Expected:

```text
enabled
```

### Verify application deployment

```bash
curl http://<MANAGED_NODE_IP>
```

### Validate Idempotency

Run the Playbook again without changing the desired configuration:

```bash
ansible-playbook playbook.yml
```

### Validate Handler

After making a valid change to the managed Nginx template:

```text
roles/webserver/templates/nginx.conf.j2
```

execute:

```bash
ansible-playbook playbook.yml
```

A changed Nginx configuration notifies the `Reload Nginx` Handler.

Running the Playbook again without another configuration change should not unnecessarily trigger the Handler.

---

# Security

Sensitive information is intentionally excluded from version control.

The repository does not contain:

- AWS PEM private keys
- SSH private keys
- AWS credentials
- Passwords or tokens
- Environment secrets

The `.gitignore` file prevents common private-key and secret-file formats from being accidentally committed.

---

# Project Outcome

This project demonstrates a practical **Ansible configuration-management workflow on AWS**.

Instead of configuring each server independently, a centralized Ansible Control Node maintains a consistent desired configuration across the managed web-server fleet.

The project combines **AWS, Linux, SSH, Ansible, Nginx, Roles, Variables, Templates, Handlers, and Idempotency** into a single hands-on DevOps implementation.
