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
    ssh-keygen -t rsa -b 2048 -f ~/.ssh/key-for-spacelift
    ```
  ![Screenshot (194)](https://github.com/user-attachments/assets/2996ed38-0f08-4aa9-a289-f14b2f9c18e0)

### 2. AWS Console Setup

- Created custom IAM roles with appropriate EC2 access.
- Attached the role ARN to the Spacelift stacks.
![Screenshot (203)](https://github.com/user-attachments/assets/7feb88ef-e1e4-4bc2-ae3b-827f15ea2ecf)
![Screenshot (204)](https://github.com/user-attachments/assets/08e5bd58-7556-41f2-b55a-7f0d2883434a)
![Screenshot (205)](https://github.com/user-attachments/assets/e2912ca9-0fc9-4d69-8e30-a651c7bdb637)
![Screenshot (206)](https://github.com/user-attachments/assets/57fb8690-397d-4d69-8b4b-7d6f84d0fa2b)
![Screenshot (207)](https://github.com/user-attachments/assets/e96f7799-2068-4c14-a24c-eff5efa090cb)
![Screenshot (210)](https://github.com/user-attachments/assets/d4b49096-a9f1-4783-999a-22614a5a1585)

### 3. Spacelift Integration (Terraform)
- Created new workspace 
![Screenshot (193)](https://github.com/user-attachments/assets/1aca8585-1796-4f38-8109-8d39f5c64c44)

- Created a new stack `terraform-stack1`.
- Set **project root** to the Terraform code folder (`terraform/`).
![Screenshot (196)](https://github.com/user-attachments/assets/e48c7172-fc9a-444a-8543-3acb89e64eab)
![Screenshot (197)](https://github.com/user-attachments/assets/78d2558b-c690-44a7-bbb2-f00cee5b260f)

- Created and attached a **context** with the SSH public key file at:
```
/mnt/workspace/key-for-spacelift.pub
```
![Screenshot (198)](https://github.com/user-attachments/assets/5121fa63-e323-4319-a0d3-3ba69d494d2e)

- Final Stack Review
![Screenshot (199)](https://github.com/user-attachments/assets/729f4eee-3ae1-4a0a-8913-8f39abe04fd8)
![Screenshot (200)](https://github.com/user-attachments/assets/9d1fa676-8402-4253-aa1b-90272b953246)

![Screenshot (202)](https://github.com/user-attachments/assets/d2385b5e-5cd8-4215-a5d3-ee7a5cbf6743)

- Set environment variable
```
TF_VAR_public_key=/mnt/workspace/key-for-spacelift.pub
```
- Triggered the stack, which:
- Initializes Terraform
- Plans infrastructure changes
- Applies changes (if auto-apply is enabled)
![Screenshot (215)](https://github.com/user-attachments/assets/f28ddea7-fda3-4586-a95e-85bd3b443c96)
![Screenshot (216)](https://github.com/user-attachments/assets/58acac6e-1b74-4b2c-b406-1bf62862f1a2)

#### ✅ Common Issue:
> If Terraform returns "No changes" but resources are missing:
- Use `terraform taint` in a custom run to force resource recreation.
- Or reset Terraform state via Stack Settings.

---

### 4. Ansible Setup

- Created an Ansible playbook `install_htop.yaml` to install `htop`.
- Pushed changes to GitHub for integration.
- Created a new stack `ansible-stack1` in Spacelift with:
![Screenshot (219)](https://github.com/user-attachments/assets/b5053d34-ab35-4d6a-99e0-5f4035e78896)


- Custom hook to set correct permissions:
  ```bash
  chmod 600 /mnt/workspace/key-for-spacelift
  ```
- Hook to extract public IPs from `terraform apply` output and write to:
  ```
  /mnt/workspace/inventory.ini
  ```
![Screenshot (221)](https://github.com/user-attachments/assets/afc72ce4-09aa-4b94-8163-c87694053c42)


### 5. Dependency Setup

- Configured the Ansible stack to be **dependent on** the Terraform stack, so it auto-triggers on infrastructure changes.
- ![Screenshot (226)](https://github.com/user-attachments/assets/b09e65ae-2e24-43b8-8ed2-bd62f05937c2)
![Screenshot (227)](https://github.com/user-attachments/assets/00ab6323-5e96-447a-b262-8129f8f03a74)


### 6. Ansible Stack Environment Variables

| Variable Name              | Value (example)                             |
|---------------------------|----------------------------------------------|
| `ansible_remote_user`     | `ubuntu`                                     |
| `ansible_private_key`     | `/mnt/workspace/key-for-spacelift`           |
| `ansible_inventory`       | `/mnt/workspace/inventory.ini`              |
![Screenshot (230)](https://github.com/user-attachments/assets/faa6dd96-7016-4665-8a36-c91f6596300f)

### 7. Testing & Validation

- Triggered Ansible stack after Terraform changes.
- Verified Ansible playbook executed and `htop` was installed on all provisioned EC2 instances.
``` Bash
ubuntu@ipaddress : htop
```
- SSH and check
![Screenshot (235)](https://github.com/user-attachments/assets/a53acb45-c7d0-4c2a-9c9a-973765d386ce)

---

## ✅ Outcome

- EC2 instances were automatically provisioned.
- SSH access was secured using Spacelift-managed keys.
- Ansible installed software on EC2 instances without manual intervention.
- Infrastructure and configuration updates were reflected immediately after pushing changes to GitHub.

---

## 📁 Repository Layout

```
terraform/
├── main.tf
├── variables.tf
ansible/
├── install_htop.yaml
├── install_ngnix.yaml
├── inventory.ini (generated by hook)
```
