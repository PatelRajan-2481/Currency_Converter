# ACS730 Final Project - Two-Tier Web Application Infrastructure with Terraform, Ansible, and GitHub Actions




## Project Author
**Name**: Rajan Patel  
**GitHub**: [PatelRajan-2481](https://github.com/PatelRajan-2481)  
**Course**: ACS730 - Cloud Infrastructure Automation  
**Section**: Winter 2025 - NBB  

---

## Project Objective
This project demonstrates an automated deployment of a two-tier web application architecture in AWS using:
- **Terraform** (Infrastructure as Code)
- **Ansible** (Configuration Management)
- **GitHub Actions** (CI/CD)

The architecture provisions and configures the following:
- 6 EC2 instances across **4 public and 2 private subnets** in multiple AZs
- NAT Gateway for internet access to private subnets
- Bastion Host for SSH access to private VMs
- Apache HTTP web servers serving static content with image loaded from **private S3 bucket**

---

## Prerequisites
Before deploying, ensure the following:

### ✅ AWS Requirements
- AWS Management Account
- One S3 bucket per environment:
  - Used for **Terraform state backend**
  - Stores an **image file (e.g. `logo.png`)** used by the webservers
- Example image path: `s3://my-bucket-name/logo.png` (uploaded manually)

### ✅ Terraform Backend Setup
Each environment (`dev`, `staging`, `prod`) must use a dedicated backend block like below:
```
terraform {
  backend "s3" {
    bucket = "acs730-rajan-dev-tfstate"
    key    = "terraform.tfstate"
    region = "us-east-1"
  }
}
```
> You must create the bucket and update `config.tf` for each environment **before** running `terraform init`.

---

## Deployment Steps

### 📁 Clone the Repository
```bash
git clone https://github.com/PatelRajan-2481/ACS730-rajan-finalproject.git
cd ACS730-rajan-finalproject
```

### ⚙️ Setup Terraform Backend (Dev Example)
```bash
cd environments/dev
terraform init
```

### 🛠️ Deploy Infrastructure
```bash
terraform apply -auto-approve
```
> Repeat for `staging/` and `prod/` directories as needed

### 📷 Upload Image to S3
Use AWS CLI or Console to upload a static image to your private S3 bucket.
```bash
aws s3 cp ./assets/index-image.png s3://acs730-rajan-web-pictures/index-image.png
```

### 📦 Install Ansible 
```bash
sudo yum install -y ansible python3-pip
pip3 install boto3 botocore
```

### ⚙️ Run Ansible Playbook
```bash
cd ACS730-rajan-finalproject
ansible-playbook ansible/playbooks/install_apache.yaml
```
> This will configure Apache on Webserver1, Webserver3 and Webserver4 using dynamic inventory

---

## Configuration Overview

### Terraform Modules
- **networking**: VPC, subnets, IGW, route tables, NAT GW
- **ec2**: Webservers, Bastion, DB, and other instances
- **security**: Security groups for public and private tiers


### Environments
- Each environment has its own:
  - `main.tf`, `config.tf`, `variables.tf`, `outputs.tf`
  - Backend state bucket
  - Naming conventions (`Rajan-Dev-Webserver1`, etc.)

### GitHub Actions
- **terraform-scan.yml**: Runs `tflint`, `tfsec`, and `trivy` on pushes to `main`, `staging`, and PRs to `prod`
- **terraform-deploy.yml**: Trigger on push to `prod`, Initializes and applies Terraform to deploy infrastructure in the `prod` branch
- **Ansible workflow**: Executes Ansible playbook on push or manual trigger
- Branch protection prevents direct pushes to `prod`


---

## 🔚 Cleanup Instructions
Run the following to delete infrastructure for an environment:
```bash
cd environments/dev
terraform destroy -auto-approve
```
> Repeat for `staging/` and `prod/` if required

Clean the local workspace:
```bash
rm -rf .terraform .terraform.lock.hcl
