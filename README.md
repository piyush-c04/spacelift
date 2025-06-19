# 🚀 Infrastructure Automation using Terraform, Spacelift & Ansible

This repository demonstrates an end-to-end infrastructure automation pipeline using **Terraform**, **Spacelift**, and **Ansible**.
It provisions EC2 instances on AWS and configures them using Ansible, all triggered and managed through Spacelift’s CI/CD capabilities.

---

## 🧰 Tools Used

- **Terraform** — Infrastructure provisioning
- **Spacelift** — CI/CD for infrastructure-as-code
- **AWS Console** — IAM role and instance access management
- **Ansible** — Post-deployment configuration (e.g., installing packages like `htop`)
- **GitHub** — Source code and trigger integration

---

## 🛠️ Steps Performed

### 1. Terraform Setup

- Wrote `main.tf` to:
  - Configure AWS provider (region: `ap-south-1`)
  - Fetch the latest Ubuntu AMI using `data "aws_ami"`
  - Define multiple EC2 instances using `local.instances`
  - Generate and reference SSH key pair using:
    ```bash
    ssh-keygen -t rsa -b 4096 -f ~/.ssh/key-for-spacelift
    ```

### 2. AWS Console Setup

- Created custom IAM roles with appropriate EC2 access.
- Attached the role ARN to the Spacelift stacks.

### 3. Spacelift Integration (Terraform)

- Created a new stack `terraform-stack1`.
- Set **project root** to the Terraform code folder (`terraform/`).
- Created and attached a **context** with the SSH public key file at:
