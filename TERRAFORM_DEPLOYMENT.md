# Terraform Deployment Guide

This guide explains how to deploy the required Red Hat Enterprise Linux (RHEL) infrastructure for the RHCE Capstone Project using Terraform, and how to subsequently configure the nodes using Ansible.

## Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/downloads) installed on your control node.
- Cloud provider credentials configured (e.g., AWS, GCP, Azure, or a local hypervisor like libvirt).
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) installed on your control node.
- SSH keys generated for node access.

## 1. Infrastructure Requirements

The project architecture requires 5 RHEL nodes:
- **1x Load Balancer Node**
- **2x Web Tier Nodes**
- **1x Database Tier Node**
- **1x Shared Storage Tier (NFS) Node**

Ensure your Terraform configurations define these 5 instances, preferably with RHEL 8 or 9 AMIs/Images, and configure the necessary security groups/firewall rules to allow SSH access from your control node, as well as internal communication between nodes.

## 2. Provisioning the Nodes

1. Initialize the Terraform workspace to download the necessary provider plugins:
   ```bash
   terraform init
   ```

2. Review the execution plan to verify the resources that will be created:
   ```bash
   terraform plan
   ```

3. Apply the Terraform configuration to provision the nodes:
   ```bash
   terraform apply
   ```
   *Type `yes` when prompted to confirm the deployment.*

## 3. Generating the Ansible Inventory

Once the nodes are provisioned, you need to map their IP addresses to the Ansible inventory.

You can use Terraform outputs or a dynamic inventory script, but for a static inventory:
1. Retrieve the IP addresses of your provisioned nodes (e.g., `terraform output`).
2. Update the `inventory/hosts.yml` file in the Ansible project with the public or private IP addresses of the respective instances.

Example `inventory/hosts.yml` mapping:
```yaml
all:
  children:
    load_balancer:
      hosts:
        lb_node:
          ansible_host: <LB_IP_ADDRESS>
    web_servers:
      hosts:
        web_node1:
          ansible_host: <WEB1_IP_ADDRESS>
        web_node2:
          ansible_host: <WEB2_IP_ADDRESS>
    database_server:
      hosts:
        db_node:
          ansible_host: <DB_IP_ADDRESS>
    storage_server:
      hosts:
        nfs_node:
          ansible_host: <NFS_IP_ADDRESS>
  vars:
    ansible_user: cloud-user
    ansible_ssh_private_key_file: /path/to/your/private/key
```

## 4. Running the Ansible Playbook

With the infrastructure provisioned and the inventory updated, you can now run the central orchestration playbook.

1. Verify Ansible can connect to all nodes:
   ```bash
   ansible all -m ping -i inventory/hosts.yml
   ```

2. Execute the central `site.yml` playbook to configure the environment:
   ```bash
   ansible-playbook -i inventory/hosts.yml site.yml
   ```
   *If your project utilizes Ansible Vault for secrets (e.g., `group_vars/all/secure.yml`), remember to include the `--ask-vault-pass` or `--vault-password-file` flag.*

## 5. Teardown

When you are finished and wish to destroy the environment, run:
```bash
terraform destroy
```
*Type `yes` when prompted to confirm the destruction of the resources.*
