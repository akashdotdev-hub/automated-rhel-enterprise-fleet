# Automated Enterprise Linux Infrastructure Deployment
### RHCE Capstone Project Submission

**Author:** Akash Babu (`akashdotdev-hub`)  
**Submission Date:** May 23, 2026  
**Target Evaluator:** shyam.s@ipsrsolutions.com  

---

## Project Overview
This project completely automates the provisioning, networking, security hardening, and deployment of a multi-tier web application infrastructure utilizing Red Hat Enterprise Linux (RHEL) nodes orchestrated from a local Ansible Control Node.

### Architecture Breakdown
* **Control Node:** Local Workstation running Ansible.
* **Load Balancer (1 Node):** HAProxy distributing incoming traffic over HTTP (Port 80).
* **Web Tier (2 Nodes):** Apache HTTPD configured with Virtual Hosts and Self-Signed SSL Certificates (HTTPS Port 443).
* **Database Tier (1 Node):** Hardened MariaDB Server hosting the relational application database schemas.
* **Shared Storage Tier (1 Node):** Centralized NFS Server providing synchronized network mounts to the Web Node cluster directory roots.

---

## Security Implementation & Hardening
* **SELinux Enforcement:** Target systems strictly maintain `enforcing` policies with customized booleans enabled for Apache-to-NFS and Database communication channels.
* **Firewalld Topography:** Minimalist port configuration strictly limits external exposure; only necessary services (HTTP, HTTPS, SSH, MariaDB, NFS) are exposed based on explicit structural rules.
* **SSH Hardening:** Direct root password authentication is explicitly blocked across all managed target environments.
* **Ansible Vault:** Application credentials and sensitive database authorization tables are securely encrypted using Vault architectures.

---

## Directory Layout
```plaintext
rhce-capstone/
├── inventory/
│   └── hosts.yml          # Cloud node inventories mapped with internal mappings
├── group_vars/
│   └──all/
        └── secure.yml         # Vault encrypted system secrets
├── roles/
│   ├── common/            # Base security configurations, firewalld, SELinux policies
│   ├── file_server/       # NFS exports setup
│   ├── db_server/         # MariaDB database setup and administrative configurations
│   ├── web_server/        # Apache engine setups, Jinja2 templates, and SSL structures
│   ├── load_balancer/     # Front-end HAProxy proxy routing configurations
│   └── maintenance/       # Nightly maintenance backup tasks and shell scripts
├── site.yml               # Central project orchestration playbook
└── README.md              # Project technical overview
