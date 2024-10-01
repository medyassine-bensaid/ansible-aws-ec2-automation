# Automating AWS EC2 Management with Ansible

This guide demonstrates how I used Ansible to automate the management of AWS EC2 instances. I have installed necessary Ansible collections, securely managed AWS credentials with Ansible Vault, defined my inventory, provisioned EC2 instances, set up passwordless SSH authentication, and implemented conditional instance management. Additionally, I have covered essential security considerations to ensure my infrastructure remains secure.

## Overview
![image](https://github.com/user-attachments/assets/a4729cb9-1200-45bf-a136-0ac1ffff8063)


## Prerequisites

- **Ansible Installed**: I ensured Ansible was installed on my local machine.
- **AWS Account**: I had an AWS account with permissions to manage EC2 instances.
- **SSH Keys**: I was familiar with SSH key generation and management.

## Steps Overview

1. [Installed the Ansible Collection for AWS](#1-installed-the-ansible-collection-for-aws)
2. [Configured AWS Credentials with Ansible Vault](#2-configured-aws-credentials-with-ansible-vault)
3. [Defined Inventory](#3-defined-inventory)
4. [Provisioned EC2 Instances](#4-provisioned-ec2-instances)
5. [Set Up Passwordless SSH Authentication](#5-set-up-passwordless-ssh-authentication)
6. [Implemented Conditional Instance Management](#6-implemented-conditional-instance-management)
7. [Verified Deployment](#7-verified-deployment)
8. [Considered Security](#8-considered-security)
9. [Concluded the Project](#9-concluded-the-project)

---

## 1. Installed the Ansible Collection for AWS

First, I installed the `amazon.aws` collection, which provided modules and plugins for interacting with AWS services.

```shell
ansible-galaxy collection install amazon.aws
```

## 2. Configured AWS Credentials with Ansible Vault

To securely manage my AWS credentials, I used Ansible Vault to encrypt sensitive information.

### Created Vault Password

I generated a strong password for Ansible Vault using OpenSSL:

```shell
openssl rand -base64 2048 > scripts/vault.pass
```

### Added AWS Credentials to Vault

I created a vaulted file to store my AWS access keys:

```shell
ansible-vault create group_vars/all/pass.yml --vault-password-file scripts/vault.pass
```

Inside the editor that opened, I added my AWS credentials in YAML format:

```yaml
ec2:
  access_key: ACCESS_KEY
  secret_key: SECRET_KEY
```

*I replaced `ACCESS_KEY` and `SECRET_KEY` with my actual AWS credentials.*

## 3. Defined Inventory

I listed my EC2 instances in an inventory file. This example used the INI format.

```ini
# inventory/inventory.ini

[all]
ec2_user@3.25.73.73
ubuntu@3.27.140.245
ubuntu@3.27.185.253
```

*I replaced the IP addresses with the public IPs of my actual EC2 instances.*

## 4. Provisioned EC2 Instances

I used an Ansible playbook to create EC2 instances. Below is how I ran the `ec2_create.yaml` playbook.

```shell
ansible-playbook playbooks/ec2_create.yaml --vault-password-file scripts/vault.pass
```

*I ensured that my `ec2_create.yaml` playbook was properly configured to provision the desired instances.*

## 5. Set Up Passwordless SSH Authentication

To allow Ansible to connect to my EC2 instances without entering passwords, I set up SSH key-based authentication.

### Generated SSH Key Pair

Since I didn't already have an SSH key pair, I generated one:

```shell
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa
```

*I followed the prompts to complete key generation, pressing Enter to accept the default file location and leaving the passphrase empty for automation purposes.*

### Copied Public Key to EC2 Instances

I transferred my public SSH key to each EC2 instance:

```shell
ssh-copy-id -i ~/.ssh/id_rsa.pub ubuntu@<INSTANCE-PUBLIC-IP>
```

*I replaced `<INSTANCE-PUBLIC-IP>` with the public IP address of each EC2 instance and repeated this step for each instance.*

## 6. Implemented Conditional Instance Management

I managed instances based on specific conditions, such as shutting down only Ubuntu instances. I used the `ec2_shutdown.yaml` playbook with conditionals.

```shell
ansible-playbook playbooks/ec2_shutdown.yaml --vault-password-file scripts/vault.pass
```

*I ensured that my `ec2_shutdown.yaml` playbook included conditionals that targeted only the desired instances based on criteria like OS family.*

## Playbook Summaries

### `playbooks/ec2_create.yaml`

This playbook provisioned three EC2 instances (two Ubuntu and one Amazon Linux) using Ansible loops. It utilized the `amazon.aws.ec2_instance` module to create instances with specified configurations.

### `playbooks/ec2_shutdown.yaml`

This playbook shut down only the Ubuntu instances by leveraging Ansible conditionals based on the OS family. It ensured that only targeted instances were affected by the shutdown operation.

## 7. Verified Deployment

After running my playbooks, I verified that everything was set up correctly.

### Checked EC2 Instances

I logged in to the AWS Management Console and navigated to the EC2 dashboard to ensure all instances were running as expected.

### Verified SSH Access

I tested SSH access to each instance to confirm that passwordless authentication was functioning correctly:

```shell
ssh ubuntu@3.27.140.245
```

*I replaced the IP address with my instance's public IP.*

### Confirmed Shutdown

After executing the shutdown playbook, I verified that only the Ubuntu instances had been stopped. I checked the instance states in the AWS Management Console or used the AWS CLI:

```shell
aws ec2 describe-instances --instance-ids i-xxxxxxxxxxxxxxxxx
```

*I replaced `i-xxxxxxxxxxxxxxxxx` with my actual instance IDs.*

## 8. Considered Security

Maintaining security is crucial when managing cloud infrastructure. Here are some best practices I implemented in this setup:

### Ansible Vault

- **Encryption**: I used Ansible Vault to encrypt sensitive data like AWS credentials, ensuring they were not exposed in plain text.
- **Vault Password Management**: I stored the vault password securely and restricted access to it.

### IAM Permissions

- **Least Privilege**: I ensured that the IAM user or role used by Ansible had only the necessary permissions to manage EC2 instances, reducing the risk of accidental or malicious actions.

### SSH Keys

- **Secure Storage**: I kept my SSH private keys secure and did not expose them in repositories or share them unnecessarily.
- **Key Rotation**: I regularly rotated SSH keys and AWS access keys to minimize potential security risks.

