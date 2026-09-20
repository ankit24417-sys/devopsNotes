# 🌍 Core Concepts & IaC Fundamentals

### 📖 Definition & History
- **Terraform** is an open-source Infrastructure as Code (IaC) tool created by **HashiCorp** in **2014**.
- It allows you to define, provision, and manage cloud and on-premise infrastructure using a high-level, human-readable configuration language called **HCL (HashiCorp Configuration Language)**.
- **Declarative Approach**: You define *what* the desired end state looks like, and Terraform automatically calculates the steps needed to reach that state.

### ❓ Why IaC Matters
- **Version Control**: Infrastructure changes are tracked in Git with commit history, pull requests, and peer reviews.
- **Eliminates Configuration Drift**: Detects manual changes made in the cloud console and restores the desired state.
- **Fast & Repeatable**: Spin up identical development, staging, and production environments in minutes.
- **Idempotency**: Running `terraform apply` multiple times produces the exact same infrastructure state without unexpected duplicates.

### ⚖️ Tool Comparison

#### Terraform vs Ansible
| Feature | Terraform | Ansible |
| :--- | :--- | :--- |
| **Primary Role** | **Infrastructure Provisioning** (creates VPCs, VMs, subnets, databases) | **Configuration Management** (installs packages, manages configs, starts services) |
| **Approach** | Declarative (defines desired end state) | Hybrid (procedural tasks executed sequentially) |
| **State Tracking**| Maintains state file (`terraform.tfstate`) | Stateless (queries live system state) |
| **Best Practice** | Use Terraform to build the infrastructure, then use Ansible to configure the OS and apps. |

#### Terraform vs AWS CloudFormation
| Feature | Terraform | AWS CloudFormation |
| :--- | :--- | :--- |
| **Cloud Support** | Multi-Cloud (AWS, Azure, GCP, Kubernetes, etc.) | AWS Only |
| **Language** | HCL (HashiCorp Configuration Language) | JSON or YAML |
| **State Storage** | Managed by user (S3, Terraform Cloud, etc.) | Fully managed automatically by AWS |

---

## 📥 Installation (Ubuntu)

Install Terraform on Ubuntu using HashiCorp's official repository:

```bash
# 1. Update system packages and install prerequisites
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl

# 2. Add HashiCorp GPG key
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

# 3. Add HashiCorp official repository
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

# 4. Update repository cache and install Terraform
sudo apt-get update && sudo apt-get install -y terraform

# 5. Verify installation
terraform --version
```

---

## 📄 HCL (HashiCorp Configuration Language) Deep Dive

HCL is designed to be simple, human-readable, and machine-friendly.

### 🧱 1. Basic Structure

Every Terraform configuration file uses standard HCL block syntax directly:

```hcl
block_type "label1" "label2" {
  argument = value
}
```

- **block_type**: The type of block (e.g., `resource`, `variable`, `provider`, `module`).
- **"label1"**: The first label defining the specific type (e.g., `"aws_instance"`).
- **"label2"**: The second label defining your custom name (e.g., `"web"`).
- **argument = value**: Arguments inside the block that configure the resource.

#### Real Example:
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  tags = {
    Name = "web"
  }
}
```

---

### 📦 2. Core Block Types

```hcl
# 1. PROVIDER: Defines which cloud or API Terraform connects to
provider "aws" {
  region = "us-east-1"
}

# 2. VARIABLE: Inbound parameters (like function arguments)
variable "instance_type" {
  type    = string
  default = "t3.micro"
}

# 3. RESOURCE: Infrastructure objects to create and manage
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type

  tags = {
    Name = "web"
  }
}

# 4. MODULE: Reusable container of multiple resources
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  name   = "my-vpc"
  cidr   = "10.0.0.0/16"
}
```

---

### 🏷️ 3. Data Types in HCL

Terraform supports simple primitive types and complex collection types:

#### 1. `string`
Text wrapped in double quotes.
```hcl
variable "region" {
  type    = string
  default = "us-east-1"
}
```

#### 2. `number`
Integers or floating-point decimals (no quotes).
```hcl
variable "server_port" {
  type    = number
  default = 8080
}
```

#### 3. `boolean` (`bool`)
Either `true` or `false` (no quotes).
```hcl
variable "enable_public_ip" {
  type    = bool
  default = true
}
```

#### 4. `list`
An ordered sequence of items of the same type, enclosed in `[...]`.
```hcl
variable "availability_zones" {
  type    = list(string)
  default = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

# Accessing an item by index (0-indexed):
# var.availability_zones[0] returns "us-east-1a"
```

#### 5. `map`
A collection of key-value pairs, enclosed in `{...}`, where all values share the same type.
```hcl
variable "instance_sizes" {
  type = map(string)
  default = {
    dev   = "t3.micro"
    stage = "t3.small"
    prod  = "t3.large"
  }
}

# Accessing a value by key:
# var.instance_sizes["dev"] returns "t3.micro"
```

#### 6. `object`
A complex type that holds multiple named attributes, each with its own distinct type.
```hcl
variable "server_profile" {
  type = object({
    hostname = string
    cpu_cores = number
    is_active = bool
  })
  default = {
    hostname  = "app-node-01"
    cpu_cores = 4
    is_active = true
  }
}

# Accessing an attribute:
# var.server_profile.hostname returns "app-node-01"
```

#### 7. `comments`
Write notes or document code using three different comment styles:
```hcl
# 1. Single-line comment using hash (most common and recommended)

// 2. Single-line comment using double slashes

/*
  3. Multi-line comment
  useful for long explanations
  or temporarily commenting out code blocks
*/
```

---

### 📥 4. Input Variables (`variable`)

Variables allow you to customize configurations without changing the source code.

#### Defining Variables:
```hcl
variable "environment" {
  type        = string
  default     = "dev"
  description = "Deployment target environment"

  # Optional validation rule:
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "db_password" {
  type        = string
  sensitive   = true # Masks the value from console output and logs
}
```

#### How to Assign Values to Variables:
1. **Default Value**: Set inside the `variable` block.
2. **Variable File (`terraform.tfvars`)**:
   ```hcl
   environment = "prod"
   ```
3. **Command Line Flag**:
   ```bash
   terraform apply -var="environment=prod"
   ```
4. **Environment Variables**:
   ```bash
   export TF_VAR_environment="prod"
   ```

---

### 📌 5. Local Values (`locals`)

Locals act like internal variables or constants. Unlike `variable` blocks (which take input from the outside), `locals` are defined and calculated **inside** the module to avoid repetitive code.

#### Defining and Using Locals:
```hcl
locals {
  app_name = "payment-service"
  env      = var.environment

  # Combining values using string interpolation:
  resource_prefix = "${local.app_name}-${local.env}"

  # Reusable common tags map:
  common_tags = {
    Application = local.app_name
    Environment = local.env
    ManagedBy   = "Terraform"
  }
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags       = local.common_tags
}
```

#### Simple Rule to Remember:
- **`variable`**: External inputs provided by the user or pipeline.
- **`locals`**: Internal shortcuts or calculated expressions computed within your code.

---

### 🔤 6. Expressions in HCL

Expressions compute or transform values dynamically:

#### 1. String Interpolation (`"${...}"`)
Combines text, variables, or resource attributes:
```hcl
name = "server-${var.environment}-${local.app_name}"
```

#### 2. Operators
- **Arithmetic**: `+`, `-`, `*`, `/`
- **Comparison**: `==`, `!=`, `<`, `>`, `<=`, `>=`
- **Logical**: `&&` (AND), `||` (OR), `!` (NOT)

#### 3. Conditional Expression (Ternary)
Selects a value based on a true/false condition:
```hcl
# condition ? true_val : false_val
instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
```

#### 4. Loops (`count` vs `for_each`)

- **`count` (Index-based loop)**:
  ```hcl
  variable "user_names" {
    type    = list(string)
    default = ["alice", "bob", "charlie"]
  }

  resource "aws_iam_user" "users" {
    count = length(var.user_names)
    name  = var.user_names[count.index]
  }
  ```

- **`for_each` (Key-based loop - Preferred)**:
  ```hcl
  variable "buckets" {
    type    = set(string)
    default = ["media", "logs", "backups"]
  }

  resource "aws_s3_bucket" "buckets" {
    for_each = var.buckets
    bucket   = "company-storage-${each.key}"
  }
  ```

#### 5. Dynamic Blocks (`dynamic`)
Used to generate repetitive nested blocks (such as firewall / security group rules):
```hcl
variable "allowed_ports" {
  type    = list(number)
  default = [80, 443, 8080]
}

resource "aws_security_group" "web_sg" {
  name   = "web-sg"
  vpc_id = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.allowed_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

---

## 🔄 Terraform Workflow & Providers

### ⚙️ Core Workflow
1. **Write**: Create or edit `.tf` configuration files.
2. **Init**: `terraform init` downloads providers, modules, and configures the backend.
3. **Plan**: `terraform plan` compares current state with desired configuration and previews changes.
4. **Apply**: `terraform apply` executes the plan and provisions cloud resources.
5. **Destroy**: `terraform destroy` tears down all managed infrastructure.

### 🚩 Important CLI Flags
- **`terraform init`**:
  - `-upgrade`: Upgrades all providers and modules to latest matching constraints.
  - `-reconfigure`: Reconfigures backend, ignoring existing state settings.
- **`terraform plan`**:
  - `-out=tfplan`: Saves the execution plan to a file to guarantee consistency.
  - `-var="key=value"`: Passes a variable directly from the CLI.
  - `-var-file="prod.tfvars"`: Loads variable values from a specific file.
- **`terraform apply`**:
  - `tfplan`: Executes a previously saved plan directly.
  - `-auto-approve`: Skips interactive confirmation (used in CI/CD pipelines).
  - `-target=resource_type.name`: Applies changes strictly to one resource.

### 🔌 Terraform Providers
Providers are plugins that translate HCL into API calls for specific platforms (AWS, Azure, GCP, Kubernetes, Docker).

#### Frequently Used Providers:
- `hashicorp/aws`
- `hashicorp/azurerm`
- `hashicorp/google`
- `hashicorp/kubernetes`

#### AWS Provider Deep Dive & Aliases (Multi-Region):
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Primary Provider
provider "aws" {
  region = "us-east-1"
  default_tags {
    tags = {
      Environment = "Production"
      ManagedBy   = "Terraform"
    }
  }
}

# Secondary Provider (Disaster Recovery region) using alias
provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

# Deploy to secondary region
resource "aws_vpc" "dr_vpc" {
  provider   = aws.west
  cidr_block = "10.1.0.0/16"
}
```

---

## ⌨️ Terraform CLI & Commands

### 📌 Core Commands
| Command | Purpose |
| :--- | :--- |
| `terraform init` | Initialize directory, download plugins and backends |
| `terraform fmt` | Format `.tf` files to standard HCL style |
| `terraform validate` | Verify syntax and internal consistency of configuration files |
| `terraform plan` | Preview changes before applying |
| `terraform apply` | Build or change infrastructure |
| `terraform destroy` | Destroy all Terraform-managed infrastructure |
| `terraform refresh` | Update state file with real-world infrastructure status |

### 🛠️ Advanced Commands
- **Taint / Replace**: Mark a resource for destruction and recreation on next apply:
  ```bash
  terraform apply -replace="aws_instance.web"
  ```
- **Import**: Bring existing cloud resources under Terraform management:
  ```bash
  terraform import aws_s3_bucket.my_bucket existing-bucket-name
  ```
- **Graph**: Generate a visual dependency graph:
  ```bash
  terraform graph | dot -Tsvg > graph.svg
  ```
- **State Manipulation**:
  ```bash
  terraform state list                       # List all resources in state
  terraform state show aws_instance.web      # Show details of a specific resource
  terraform state mv aws_instance.old aws_instance.new  # Rename resource in state
  terraform state rm aws_instance.web        # Remove from state without deleting from cloud
  ```

### 🐞 Debugging Terraform Issues
Enable verbose logging using the `TF_LOG` environment variable:
```bash
# Log levels: TRACE, DEBUG, INFO, WARN, ERROR
export TF_LOG=DEBUG
export TF_LOG_PATH="./terraform-debug.log"

terraform apply

# Disable logging
unset TF_LOG
unset TF_LOG_PATH
```

---

## 🗄️ State Management & Remote Backends

### 📌 Role of State (`terraform.tfstate`)
- Maps resources in code to real-world cloud IDs.
- Tracks resource dependencies and metadata.
- Acts as a performance cache to minimize cloud API requests.

### 🔒 Secure State Best Practices
- **Never commit `.tfstate` to Git**: State files contain sensitive plain-text values (passwords, tokens). Add `*.tfstate` to `.gitignore`.
- **Use Remote Encrypted Storage**: Store state in remote backends (e.g., AWS S3 with server-side encryption enabled).
- **Enable State Locking**: Use a lock mechanism (e.g., DynamoDB) to prevent concurrent executions from corrupting the state.

### ☁️ Remote Backend: AWS S3 + DynamoDB
```hcl
terraform {
  backend "s3" {
    bucket         = "company-terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-locks"
  }
}
```

---

## ⚡ Provisioners vs User Data

### Understanding Provisioners
Provisioners run scripts or copy files during resource creation:
- **`file`**: Copies files from local machine to remote resource.
- **`remote-exec`**: Runs shell commands on remote resource over SSH/WinRM.
- **`local-exec`**: Runs commands on the local machine executing Terraform.

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  provisioner "local-exec" {
    command = "echo ${self.public_ip} >> instance_ips.txt"
  }
}
```

### ⚠️ Why Provisioners are a Last Resort
- Provisioners break the declarative model and cannot detect drift.
- They require open SSH ports and live network connections during provisioning.
- **Preferred Alternatives**:
  - **`user_data` / Cloud-init**: Bootstraps the VM natively during initial launch.
  - **Packer**: Pre-build AMIs with all dependencies pre-installed.
  - **Ansible / SSM**: Use configuration management tools post-provisioning.

```hcl
# The Recommended Way: Using user_data
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  user_data = <<-EOF
              #!/bin/bash
              apt-get update -y
              apt-get install -y nginx
              systemctl enable --now nginx
              EOF
}
```

---

## 🗂️ Workspaces & Environment Management

### What are Workspaces?
Workspaces allow multiple distinct state files to be associated with a single configuration directory.

### Commands:
```bash
terraform workspace list          # List available workspaces
terraform workspace new dev       # Create 'dev' workspace
terraform workspace new prod      # Create 'prod' workspace
terraform workspace select dev    # Switch to 'dev' workspace
terraform workspace show          # Print current workspace name
```

### Example Usage:
```hcl
resource "aws_instance" "server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = terraform.workspace == "prod" ? "m5.large" : "t3.micro"

  tags = {
    Name        = "server-${terraform.workspace}"
    Environment = terraform.workspace
  }
}
```

### Workspaces vs Separate Directories
- **Workspaces**: Best for testing, feature branches, and ephemeral environments with identical setups.
- **Separate Directories (`/dev`, `/prod`)**: Industry standard for production. Provides complete isolation of state, credentials, and configurations.

---

## 🧩 Terraform Modules (Reusability & Best Practices)

### Module Types
- **Root Module**: The primary working directory where `terraform` commands are executed.
- **Child Module**: Any module called into a configuration using a `module` block.

### Standard Module Directory Structure:
```
modules/ec2-instance/
├── README.md        # Documentation
├── main.tf          # Resource declarations
├── variables.tf     # Input variables
├── outputs.tf       # Exported values
└── versions.tf      # Provider version constraints
```

### Calling a Custom Module:
```hcl
module "app_server" {
  source        = "./modules/ec2-instance"
  instance_name = "frontend-web"
  instance_type = "t3.small"
}

output "app_ip" {
  value = module.app_server.public_ip
}
```

---

## 🚀 Hands-On Project: AWS EKS Setup

### Step 1: VPC, Subnets & Route Tables
```hcl
# network.tf
resource "aws_vpc" "eks_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "eks-vpc"
  }
}

# Public Subnets (for NAT Gateway & Load Balancers)
resource "aws_subnet" "public_1" {
  vpc_id                  = aws_vpc.eks_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true

  tags = {
    Name                     = "eks-public-1a"
    "kubernetes.io/role/elb" = "1"
  }
}

# Private Subnets (for EKS Worker Nodes)
resource "aws_subnet" "private_1" {
  vpc_id            = aws_vpc.eks_vpc.id
  cidr_block        = "10.0.10.0/24"
  availability_zone = "us-east-1a"

  tags = {
    Name                              = "eks-private-1a"
    "kubernetes.io/role/internal-elb" = "1"
  }
}

resource "aws_subnet" "private_2" {
  vpc_id            = aws_vpc.eks_vpc.id
  cidr_block        = "10.0.20.0/24"
  availability_zone = "us-east-1b"

  tags = {
    Name                              = "eks-private-1b"
    "kubernetes.io/role/internal-elb" = "1"
  }
}

# Internet Gateway & NAT Gateway
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.eks_vpc.id
}

resource "aws_eip" "nat" {
  domain = "vpc"
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public_1.id
}

# Route Tables
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.eks_vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.eks_vpc.id
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }
}

resource "aws_route_table_association" "pub_assoc" {
  subnet_id      = aws_subnet.public_1.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "priv_assoc_1" {
  subnet_id      = aws_subnet.private_1.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "priv_assoc_2" {
  subnet_id      = aws_subnet.private_2.id
  route_table_id = aws_route_table.private.id
}
```

### Step 2: IAM Roles & Policies for EKS
```hcl
# iam.tf
# 1. Cluster IAM Role
resource "aws_iam_role" "eks_cluster_role" {
  name = "eks-cluster-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Effect    = "Allow"
      Principal = { Service = "eks.amazonaws.com" }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "cluster_policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
  role       = aws_iam_role.eks_cluster_role.name
}

# 2. Node Group IAM Role
resource "aws_iam_role" "eks_nodes_role" {
  name = "eks-nodes-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action    = "sts:AssumeRole"
      Effect    = "Allow"
      Principal = { Service = "ec2.amazonaws.com" }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "worker_node_policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"
  role       = aws_iam_role.eks_nodes_role.name
}

resource "aws_iam_role_policy_attachment" "cni_policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
  role       = aws_iam_role.eks_nodes_role.name
}

resource "aws_iam_role_policy_attachment" "ecr_policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
  role       = aws_iam_role.eks_nodes_role.name
}
```

### Step 3: Deploying EKS Cluster & Node Group
```hcl
# eks.tf
resource "aws_eks_cluster" "eks" {
  name     = "main-eks-cluster"
  role_arn = aws_iam_role.eks_cluster_role.arn
  version  = "1.29"

  vpc_config {
    subnet_ids = [aws_subnet.private_1.id, aws_subnet.private_2.id]
  }

  depends_on = [aws_iam_role_policy_attachment.cluster_policy]
}

resource "aws_eks_node_group" "nodes" {
  cluster_name    = aws_eks_cluster.eks.name
  node_group_name = "general-nodes"
  node_role_arn   = aws_iam_role.eks_nodes_role.arn
  subnet_ids      = [aws_subnet.private_1.id, aws_subnet.private_2.id]

  scaling_config {
    desired_size = 2
    max_size     = 4
    min_size     = 1
  }

  instance_types = ["t3.medium"]

  depends_on = [
    aws_iam_role_policy_attachment.worker_node_policy,
    aws_iam_role_policy_attachment.cni_policy,
    aws_iam_role_policy_attachment.ecr_policy
  ]
}
```

---

## 🤖 CI/CD with Terraform

### GitHub Actions Workflow (`.github/workflows/terraform.yml`)
```yaml
name: "Terraform CI/CD"

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.7.0

      - name: Terraform Format Check
        run: terraform fmt -check

      - name: Terraform Init
        run: terraform init

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan (Pull Request)
        if: github.event_name == 'pull_request'
        run: terraform plan

      - name: Terraform Apply (Main Branch Only)
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: terraform apply -auto-approve
```

### Jenkins Pipeline (`Jenkinsfile`)
```groovy
pipeline {
    agent any

    stages {
        stage('Init & Validate') {
            steps {
                sh 'terraform init -reconfigure'
                sh 'terraform validate'
            }
        }

        stage('Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
            }
        }

        stage('Approval Gate') {
            steps {
                input message: 'Approve applying Terraform plan to infrastructure?'
            }
        }

        stage('Apply') {
            steps {
                sh 'terraform apply -auto-approve tfplan'
            }
        }
    }
}
```

---

## 🤝 Terraform with Ansible (Multi-Environment)

Terraform provisions the infrastructure and generates an inventory file, while Ansible performs the operating system configuration.

### 1. Generating Ansible Inventory from Terraform
```hcl
resource "aws_instance" "web" {
  count         = 2
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  tags          = { Name = "web-${count.index}" }
}

# Automatically create inventory file for Ansible
resource "local_file" "ansible_inventory" {
  content = templatefile("${path.module}/inventory.tmpl", {
    web_ips = aws_instance.web[*].public_ip
  })
  filename = "${path.module}/ansible/inventory.ini"
}
```

### 2. Inventory Template (`inventory.tmpl`)
```ini
[webservers]
%{ for ip in web_ips ~}
${ip} ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
%{ endfor ~}
```

### 3. Execution Command
```bash
terraform apply -auto-approve
ansible-playbook -i ansible/inventory.ini ansible/configure-web.yml
```
