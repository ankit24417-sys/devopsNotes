# 🌍 Core Concepts & IaC Fundamentals

### 📖 Definition & History

- **Terraform** is an open-source Infrastructure as Code (IaC) tool created by **HashiCorp** in **2014**.
- It allows you to define, provision, and manage cloud and on-premise infrastructure using a high-level, human-readable configuration language called **HCL (HashiCorp Configuration Language)**.
- **Declarative Approach**: You define _what_ the desired end state looks like, and Terraform automatically calculates the steps needed to reach that state.

### ❓ Why IaC Matters

- **Version Control**: Infrastructure changes are tracked in Git with commit history, pull requests, and peer reviews.
- **Eliminates Configuration Drift**: Detects manual changes made in the cloud console and restores the desired state.
- **Fast & Repeatable**: Spin up identical development, staging, and production environments in minutes.
- **Idempotency**: Running `terraform apply` multiple times produces the exact same infrastructure state without unexpected duplicates.

### ⚖️ Tool Comparison

#### Terraform vs Ansible

| Feature            | Terraform                                                                                 | Ansible                                                                            |
| :----------------- | :---------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------- |
| **Primary Role**   | **Infrastructure Provisioning** (creates VPCs, VMs, subnets, databases)                   | **Configuration Management** (installs packages, manages configs, starts services) |
| **Approach**       | Declarative (defines desired end state)                                                   | Hybrid (procedural tasks executed sequentially)                                    |
| **State Tracking** | Maintains state file (`terraform.tfstate`)                                                | Stateless (queries live system state)                                              |
| **Best Practice**  | Use Terraform to build the infrastructure, then use Ansible to configure the OS and apps. |

#### Terraform vs AWS CloudFormation

| Feature           | Terraform                                       | AWS CloudFormation                 |
| :---------------- | :---------------------------------------------- | :--------------------------------- |
| **Cloud Support** | Multi-Cloud (AWS, Azure, GCP, Kubernetes, etc.) | AWS Only                           |
| **Language**      | HCL (HashiCorp Configuration Language)          | JSON or YAML                       |
| **State Storage** | Managed by user (S3, Terraform Cloud, etc.)     | Fully managed automatically by AWS |

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

# 5. OUTPUT: Return values / exported resource attributes
output "instance_ip" {
  value       = aws_instance.web.public_ip
  description = "Public IP of the web server"
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

### 📤 6. Output Values (`output`) & Directly Accessing as Environment Variables

Outputs are like return values in programming languages. When Terraform provisions resources, attributes like public IPs, DNS names, resource IDs, or generated passwords can be exposed via `output` blocks.

You can directly access these Terraform outputs as **shell environment variables** for deployment scripts, SSH commands, configuration files, and CI/CD pipelines.

#### 1. Defining Outputs (`outputs.tf`):

```hcl
output "ec2-public-ip" {
  description = "Public IP address of the provisioned EC2 instance"
  value       = aws_instance.ec2_instance.public_ip
}

output "ec2-public-dns" {
  description = "Public DNS of the EC2 instance"
  value       = aws_instance.ec2_instance.public_dns
}

output "ec2-private-ip" {
  description = "Private IP address for internal VPC networking"
  value       = aws_instance.ec2_instance.private_ip
}

output "db_password" {
  description = "Sensitive database master password"
  value       = aws_db_instance.db.password
  sensitive   = true # Masks value from stdout in terraform plan and apply
}
```

#### 2. Querying Outputs via Terraform CLI:

```bash
# Print all outputs with their values
terraform output

# Print a specific output (wraps string in quotes, e.g. "54.210.12.34")
terraform output ec2-public-ip

# Print the RAW value without quotes or color codes (essential for scripts & env vars)
terraform output -raw ec2-public-ip

# Export all outputs as structured JSON (ideal for jq and automation tools)
terraform output -json
```

#### 3. Directly Accessing & Exporting Outputs as Environment Variables

##### 🔹 Method 1: Export a Single Output into a Shell Environment Variable
Always use the `-raw` flag to strip quotes so the variable can be directly used in commands without quote errors:

```bash
# Export individual output directly into an environment variable
export EC2_PUBLIC_IP=$(terraform output -raw ec2-public-ip)
export EC2_PUBLIC_DNS=$(terraform output -raw ec2-public-dns)

# Verify the variable in your current terminal session
echo "EC2 Public IP is: $EC2_PUBLIC_IP"

# Use directly in SSH or other shell commands:
ssh -i ~/.ssh/id_rsa ubuntu@$EC2_PUBLIC_IP
```

> **⚠️ Why `-raw` is Critical**:
> - Without `-raw`: `export IP=$(terraform output ec2-public-ip)` sets `IP` to `"54.210.12.34"` (with quotes). Running `ssh ubuntu@$IP` fails because the shell passes literal quote marks.
> - With `-raw`: `export IP=$(terraform output -raw ec2-public-ip)` sets `IP` to `54.210.12.34` (clean string).

##### 🔹 Method 2: Inline Environment Variables / Direct Command Substitution
If you don't need to persist the variable in your shell session, pass it inline to a command or script:

```bash
# Pass inline as an environment variable to a deployment script
HOST_IP=$(terraform output -raw ec2-public-ip) ./deploy.sh

# Direct command substitution without saving to any variable
ssh -i ssh_key ubuntu@$(terraform output -raw ec2-public-ip)
curl "http://$(terraform output -raw ec2-public-dns):8080/health"
```

##### 🔹 Method 3: Dynamically Export ALL Outputs as Environment Variables (Bulk Export)
Instead of writing an `export` command for each output individually, you can automatically parse `terraform output -json` with `jq` and export all outputs into your shell in uppercase format (converting hyphens `-` to underscores `_`):

```bash
# Dynamically export all outputs to uppercase shell environment variables
eval "$(terraform output -json | jq -r 'to_entries[] | "export \(.key | ascii_upcase | gsub("-"; "_"))=\"\(.value.value)\""')"

# Verify exported environment variables:
echo $EC2_PUBLIC_IP
echo $EC2_PUBLIC_DNS
echo $EC2_PRIVATE_IP
```

##### 🔹 Method 4: Automatically Generate a `.env` File from Terraform (`local_file`)
You can have Terraform write environment variables directly into a `.env` file upon `terraform apply`, allowing your local shell, Docker Compose, or backend apps (`dotenv`) to read them immediately:

```hcl
# In your terraform configuration (e.g. outputs.tf or main.tf):
resource "local_file" "environment_variables" {
  filename = "${path.module}/.env"
  content  = <<-EOT
    # Auto-generated by Terraform - DO NOT EDIT MANUALLY
    EC2_PUBLIC_IP=${aws_instance.ec2_instance.public_ip}
    EC2_PUBLIC_DNS=${aws_instance.ec2_instance.public_dns}
    EC2_PRIVATE_IP=${aws_instance.ec2_instance.private_ip}
  EOT
}
```

Load the generated `.env` file directly into your current shell session:

```bash
# Export all variables from .env to the current shell:
export $(grep -v '^#' .env | xargs)

# Or in bash/zsh:
set -a && source .env && set +a
```

##### 🔹 Method 5: Accessing Sensitive Outputs as Environment Variables
When an output has `sensitive = true`, running `terraform output` masks it with `<sensitive>` to prevent leaks in logs. You can still extract its real value directly into an environment variable using `-raw`:

```bash
# Sensitive values are hidden in regular output:
# terraform output db_password -> (sensitive value)

# -raw extracts the real secret into an environment variable directly:
export DB_PASSWORD=$(terraform output -raw db_password)
```

##### 🔹 Method 6: Setting Environment Variables from Outputs in CI/CD

- **GitHub Actions**: Export Terraform outputs directly into `$GITHUB_ENV` so subsequent workflow steps can use them as native environment variables:
  ```yaml
  - name: Export Terraform Outputs to GitHub Environment
    run: |
      echo "EC2_PUBLIC_IP=$(terraform output -raw ec2-public-ip)" >> $GITHUB_ENV
      echo "EC2_PUBLIC_DNS=$(terraform output -raw ec2-public-dns)" >> $GITHUB_ENV

  - name: Use Exported Environment Variables
    run: |
      echo "Connecting to $EC2_PUBLIC_IP"
      ssh -o StrictHostKeyChecking=no ubuntu@$EC2_PUBLIC_IP "systemctl status nginx"
  ```

- **GitLab CI**: Export to a dotenv artifact report:
  ```yaml
  terraform_outputs:
    stage: provision
    script:
      - echo "EC2_PUBLIC_IP=$(terraform output -raw ec2-public-ip)" >> deploy.env
    artifacts:
      reports:
        dotenv: deploy.env
  ```

#### ⚖️ Summary: Input Variables vs. Output Environment Variables

| Feature | Input Environment Variables | Output Environment Variables |
| :--- | :--- | :--- |
| **Direction** | Shell / OS $\longrightarrow$ Terraform | Terraform $\longrightarrow$ Shell / OS |
| **Mechanism** | Prefix environment variable with `TF_VAR_` | Use `terraform output -raw <name>` |
| **Example** | `export TF_VAR_ec2_name="ankit-instance"` | `export EC2_IP=$(terraform output -raw ec2-public-ip)` |
| **Primary Purpose** | Provide parameters to customize resource creation | Extract created infrastructure attributes for runtime use |

---

### 🔤 7. Expressions in HCL

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

| Command              | Purpose                                                       |
| :------------------- | :------------------------------------------------------------ |
| `terraform init`     | Initialize directory, download plugins and backends           |
| `terraform fmt`      | Format `.tf` files to standard HCL style                      |
| `terraform validate` | Verify syntax and internal consistency of configuration files |
| `terraform plan`     | Preview changes before applying                               |
| `terraform apply`    | Build or change infrastructure                                |
| `terraform destroy`  | Destroy all Terraform-managed infrastructure                  |
| `terraform refresh`  | Update state file with real-world infrastructure status       |
| `terraform output`   | Extract and display output values from state                  |

### 🛠️ Advanced Commands

- **Taint / Replace**: Mark a resource for destruction and recreation on next apply:
  ```bash
  terraform apply -replace="aws_instance.web"
  ```
- **Import**: Bring existing cloud resources under Terraform management (via CLI `terraform import` or declarative `import` block; see [full guide below](#-importing-existing-infrastructure-terraform-import)):
  ```bash
  terraform import aws_instance.web i-0123456789abcdef0
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

### 🔐 Backend State Locking Deep Dive

#### ❓ What is State Locking & Why is it Critical?

When multiple team members or automated CI/CD pipelines run Terraform concurrently against the same environment, a **race condition** can occur:

```
Developer A: Runs 'terraform apply' (reading & modifying state)
                 ⬇️ (Concurrent Execution)
Developer B / CI: Runs 'terraform apply' (overwriting state simultaneously)
                 💥 STATE CORRUPTION / RACE CONDITION!
```

Without locking:
1. **Lost Updates**: One engineer's changes overwrite the other's, leading to unmanaged or orphaned cloud resources.
2. **State Corruption**: Partial or interrupted state writes leave `terraform.tfstate` malformed, breaking subsequent runs.
3. **Duplicate Infrastructure**: Both runners may see that a resource does not exist and simultaneously issue API requests to create duplicate VPCs, subnets, or VMs.

**State Locking** solves this by acquiring an exclusive lock before any write or state-altering operation (`terraform plan`, `apply`, `destroy`), holding the lock during execution, and automatically releasing it once the operation completes.

---

#### 🔄 How State Locking Works (The Lock Lifecycle)

```
1. Engineer runs 'terraform apply'
        │
        ▼
2. Terraform contacts Backend (e.g., DynamoDB)
        │
        ├─► Is state already locked?
        │     ├─► YES: Abort with "Error acquiring the state lock" (prints Lock ID & user)
        │     └─► NO:  Write Lock record with UUID, User, Timestamp, Operation
        ▼
3. Terraform acquires lock and executes Plan & Apply
        │
        ▼
4. Terraform writes updated state to remote storage (e.g., S3)
        │
        ▼
5. Terraform deletes the Lock record from Backend (Releases Lock)
```

---

#### ☁️ 1. AWS S3 + DynamoDB State Locking (The Classic Standard)

In AWS, the S3 backend relies on an Amazon DynamoDB table to maintain distributed locks.

##### Backend Configuration:
```hcl
terraform {
  backend "s3" {
    bucket         = "company-terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-locks" # Name of DynamoDB table for locking
  }
}
```

##### 🏗️ How to Bootstrap the S3 Bucket & DynamoDB Lock Table:
> [!IMPORTANT]
> The DynamoDB table **must** have a Primary Partition Key named **`LockID`** with type **String (`S`)**. Without this exact attribute name, Terraform locking will fail.

Here is the production-ready Terraform bootstrap configuration (`backend-bootstrap.tf`):

```hcl
provider "aws" {
  region = "us-east-1"
}

# 1. S3 Bucket for State Storage
resource "aws_s3_bucket" "terraform_state" {
  bucket        = "company-terraform-state-bucket"
  force_destroy = false # Prevent accidental deletion of state bucket

  lifecycle {
    prevent_destroy = true
  }
}

# 2. Enable S3 Bucket Versioning (Essential for State Rollbacks)
resource "aws_s3_bucket_versioning" "terraform_state_versioning" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

# 3. Enable Server-Side Encryption (AES256)
resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state_crypto" {
  bucket = aws_s3_bucket.terraform_state.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# 4. Block All Public Access to the State Bucket
resource "aws_s3_bucket_public_access_block" "terraform_state_public_block" {
  bucket = aws_s3_bucket.terraform_state.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# 5. DynamoDB Table for Distributed State Locking
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-state-locks"
  billing_mode = "PAY_PER_REQUEST" # Cost-effective on-demand billing
  hash_key     = "LockID"           # MUST be exactly "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  tags = {
    Name        = "terraform-state-locks"
    Environment = "global"
    ManagedBy   = "Terraform"
  }
}
```

##### 🔍 What Does a DynamoDB Lock Entry Look Like?
When Terraform locks the state, it writes an item into DynamoDB:
- **`LockID`**: `company-terraform-state-bucket/prod/terraform.tfstate-md5`
- **`Info`**: JSON string containing execution details:
  ```json
  {
    "ID": "41cf432d-304b-7419-f53e-51c070d6741b",
    "Operation": "OperationTypeApply",
    "Info": "",
    "Who": "ankit@workstation",
    "Version": "1.7.0",
    "Created": "2026-09-27T17:15:30.123456Z",
    "Path": "company-terraform-state-bucket/prod/terraform.tfstate"
  }
  ```

---

#### 🚀 2. Modern Native S3 Locking (Terraform 1.10+)

Starting with **Terraform v1.10.0**, Terraform natively supports **S3 conditional writes** (`PutObject` with `If-None-Match`). This enables state locking directly inside S3 without needing a DynamoDB table!

```hcl
terraform {
  backend "s3" {
    bucket       = "company-terraform-state-bucket"
    key          = "prod/terraform.tfstate"
    region       = "us-east-1"
    encrypt      = true
    use_lockfile = true # Enables native S3 locking without DynamoDB!
  }
}
```

---

#### 🌐 3. State Locking Across Other Providers

State locking is supported natively across all major cloud providers:

| Backend Provider | Mechanism | Requires Extra Resource? |
| :--- | :--- | :--- |
| **AWS S3** | DynamoDB table (or native S3 lockfile in v1.10+) | DynamoDB table (optional in 1.10+) |
| **Azure Blob (`azurerm`)** | Native Blob Storage Leases | ❌ None (automatic via Azure Blob API) |
| **Google Cloud (`gcs`)** | Native Cloud Storage Object Preconditions | ❌ None (automatic via generation match) |
| **Terraform Cloud / Enterprise** | Native Workspace Run Queuing | ❌ None (fully managed) |
| **Local Backend** | System file lock on `terraform.tfstate` | ❌ None (automatic local file lock) |

---

#### 🚨 What Happens When State Is Locked? (The Lock Error)

If you or a CI/CD pipeline attempt to run Terraform while another operation holds the lock, Terraform aborts and displays:

```text
Error: Error acquiring the state lock

Error message: ConditionalCheckFailedException: The conditional request failed
Lock Info:
  ID:        41cf432d-304b-7419-f53e-51c070d6741b
  Path:      company-terraform-state-bucket/prod/terraform.tfstate
  Operation: OperationTypeApply
  Who:       ankit@workstation
  Version:   1.7.0
  Created:   2026-09-27 17:15:30.123456Z
  Info:      

Terraform acquires a state lock to protect the state from being written
by multiple users at the same time. Please resolve the issue above and try
again. For most commands, you can disable locking with the "-lock=false"
flag, but this is not recommended.
```

---

#### 🛠️ Troubleshooting: Handling Stuck Locks (`terraform force-unlock`)

##### Why Do Locks Get Stuck?
Normally, Terraform releases the lock automatically when it finishes. However, locks can remain orphaned/stuck if:
1. A CI/CD runner crashed or timed out abruptly.
2. The process was forcibly killed with `kill -9` (`SIGKILL`) instead of graceful `SIGINT` (`Ctrl+C`).
3. Network connection or power was suddenly interrupted during `terraform apply`.

##### 🔹 Solution 1: `terraform force-unlock` (Standard Fix)
Use the unique **Lock ID** from the error message to safely clear the lock:

```bash
# Syntax: terraform force-unlock <LOCK_ID>
terraform force-unlock 41cf432d-304b-7419-f53e-51c070d6741b
```

Terraform prompts for confirmation:
```text
Do you really want to force-unlock?
  Terraform will remove the lock on the remote state.
  This will allow local and remote processes to continue.

  Enter a value: yes

Terraform state has been successfully unlocked!
```

> [!CAUTION]
> **Never force-unlock blindly!**
> Always verify with your team or CI/CD dashboards that **no other engineer or pipeline is actively running `terraform apply`**. Force-unlocking a running apply can corrupt your state file.

##### 🔹 Solution 2: Manual Removal via DynamoDB (If CLI Fails)
If credentials or CLI cannot clear the lock:
1. Navigate to **AWS Console** $\rightarrow$ **DynamoDB** $\rightarrow$ **Explore Items**.
2. Select your `terraform-state-locks` table.
3. Locate the item whose `LockID` matches your bucket/key path.
4. Delete the item manually.

---

#### 🎛️ Useful CLI Flags for State Locking

- **`-lock-timeout=<duration>`**:
  Tells Terraform to wait and retry acquiring the lock for a specified time before giving up. Highly recommended in CI/CD pipelines:
  ```bash
  # Wait up to 5 minutes for a running job to release its lock:
  terraform apply -lock-timeout=5m
  ```

- **`-lock=false` (Use with Extreme Caution)**:
  Bypasses state locking completely:
  ```bash
  # Danger: only use for emergency read-only inspections if locks are broken:
  terraform plan -lock=false
  ```

---

## 📥 Importing Existing Infrastructure (Terraform Import)

### ❓ What is Resource Importing & Why is it Needed?

Often, cloud resources are created manually via the AWS Management Console, through CLI scripts, or by a legacy setup before adopting Terraform. 

**Importing** is the process of bringing existing, unmanaged real-world infrastructure into Terraform's management (both the **state file** and the **HCL configuration**) without deleting, rebuilding, or causing downtime to live resources.

> [!IMPORTANT]
> **Key Rule of Terraform Import**:
> Historically, Terraform only imported the resource into the **state file (`terraform.tfstate`)**, but did **not** automatically generate the `.tf` configuration files. You had to manually write the HCL code to match the state.
> Starting with **Terraform 1.5+**, Terraform introduced the **declarative `import` block** combined with automatic HCL code generation (`-generate-config-out`), making imports automated, previewable in plan, and safe for GitOps/CI-CD workflows!

---

### ⚖️ The Two Import Approaches

| Feature | Modern Declarative Approach (Terraform 1.5+) | Traditional CLI Approach (`terraform import`) |
| :--- | :--- | :--- |
| **Introduced** | Terraform v1.5.0+ | All Terraform versions |
| **Method** | Declarative `import { ... }` block in `.tf` file | Imperative command in shell terminal |
| **Code Generation** | **Automatic** (`-generate-config-out=...`) | **Manual** (Must write HCL code yourself) |
| **Plan Preview** | Standard `terraform plan` shows import preview | No dry-run preview; modifies state immediately |
| **Team / CI/CD** | Tracked in Git, peer-reviewed via Pull Request | Ad-hoc local execution, risks state drift |
| **Recommended?** |  **Yes (Modern Standard)** |  Legacy / Quick local one-offs |

---

### 🛠️ Hands-On Walkthrough: Importing a Manually Created EC2 Instance

**Scenario**: You manually launched an EC2 instance in the AWS Console with:
- **Instance ID**: `i-0123456789abcdef0`
- **Region**: `us-east-1`
- **Type**: `t3.micro`
- **AMI**: `ami-0c55b159cbfafe1f0`
- **Name Tag**: `manual-web-server`

---

#### 🌟 Method 1: The Modern Declarative Way (Terraform 1.5+ with Code Generation) — Recommended

This method allows Terraform to query AWS and **generate the HCL configuration for you**.

##### Step 1: Define the `import` Block
Create or open a `.tf` file (e.g., `imports.tf` or `main.tf`):

```hcl
provider "aws" {
  region = "us-east-1"
}

# 1. Define the import block:
import {
  # 'to' specifies the target Terraform resource address
  to = aws_instance.web_server

  # 'id' is the real-world cloud resource ID (EC2 Instance ID from AWS Console)
  id = "i-0123456789abcdef0"
}
```

##### Step 2: Auto-Generate HCL Code with `terraform plan`
Run `terraform plan` with the `-generate-config-out` flag to have Terraform automatically construct the `.tf` file:

```bash
terraform init
terraform plan -generate-config-out=generated_ec2.tf
```

Terraform will query AWS, inspect the instance attributes, and generate `generated_ec2.tf`:
```text
aws_instance.web_server: Preparing import... [id=i-0123456789abcdef0]
aws_instance.web_server: Refreshing state... [id=i-0123456789abcdef0]

Terraform will perform the following actions:

  # aws_instance.web_server will be imported
    resource "aws_instance" "web_server" {
        ami                          = "ami-0c55b159cbfafe1f0"
        arn                          = "arn:aws:ec2:us-east-1:123456789012:instance/i-0123456789abcdef0"
        associate_public_ip_address  = true
        availability_zone            = "us-east-1a"
        instance_type                = "t3.micro"
        key_name                     = "my-ssh-key"
        subnet_id                    = "subnet-0a1b2c3d4e5f6g7h8"
        vpc_security_group_ids       = ["sg-0123456789abcdef0"]
        tags                         = {
            "Name" = "manual-web-server"
        }
    }

Plan: 1 to import, 0 to add, 0 to change, 0 to destroy.
```

##### Step 3: Review and Clean Up `generated_ec2.tf`
Open `generated_ec2.tf` and clean up computed, read-only, or default attributes (like `arn`, `id`, `instance_state`, or hardcoded private IPs) so your configuration remains clean and maintainable:

```hcl
# generated_ec2.tf
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  key_name      = "my-ssh-key"
  subnet_id     = "subnet-0a1b2c3d4e5f6g7h8"
  vpc_security_group_ids = [
    "sg-0123456789abcdef0"
  ]

  tags = {
    Name        = "manual-web-server"
    Environment = "production"
    ManagedBy   = "Terraform"
  }
}
```

##### Step 4: Verify Plan (Zero Drift)
Run standard `terraform plan` without the generation flag:

```bash
terraform plan
```
Ensure the plan confirms:
```text
Plan: 1 to import, 0 to add, 0 to change, 0 to destroy.
```

##### Step 5: Apply the Import
```bash
terraform apply
```

The instance `i-0123456789abcdef0` is now bound into `terraform.tfstate`. Your EC2 instance is now managed by Terraform without any downtime!

> [!TIP]
> **What to do with the `import` block after apply?**
> You can safely leave the `import` block in your code (it acts as documentation of the resource origin and will not re-import on subsequent applies), or you can delete it once the import is complete.

---

#### 🏛️ Method 2: The Traditional CLI Command Way (`terraform import`)

Use this method if you are on Terraform versions prior to 1.5 or prefer running a quick CLI command.

##### Step 1: Write a Skeleton Resource in `main.tf`
Terraform requires a target resource block in your `.tf` file before running the CLI import:

```hcl
provider "aws" {
  region = "us-east-1"
}

# Skeleton resource block (initially with minimal attributes)
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}
```

##### Step 2: Run the `terraform import` Command
Execute the import command by passing the target resource address and the AWS Instance ID:

```bash
# Syntax: terraform import <RESOURCE_TYPE>.<RESOURCE_NAME> <CLOUD_RESOURCE_ID>
terraform import aws_instance.web_server i-0123456789abcdef0
```

Console output:
```text
aws_instance.web_server: Importing from ID "i-0123456789abcdef0"...
aws_instance.web_server: Import prepared!
  Prepared aws_instance for import
aws_instance.web_server: Refreshing state... [id=i-0123456789abcdef0]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
```

> [!WARNING]
> At this stage, the instance is in `terraform.tfstate`, but **your `main.tf` does not yet match the live configuration!** If you run `terraform apply` now, Terraform may attempt to modify or destroy/re-create the instance to match your bare skeleton!

##### Step 3: Inspect the Imported State with `terraform state show`
Inspect the exact configuration recorded in the state file:

```bash
terraform state show aws_instance.web_server
```

Output:
```hcl
# aws_instance.web_server:
resource "aws_instance" "web_server" {
    ami                          = "ami-0c55b159cbfafe1f0"
    arn                          = "arn:aws:ec2:us-east-1:123456789012:instance/i-0123456789abcdef0"
    associate_public_ip_address  = true
    availability_zone            = "us-east-1a"
    instance_type                = "t3.micro"
    key_name                     = "my-ssh-key"
    subnet_id                    = "subnet-0a1b2c3d4e5f6g7h8"
    vpc_security_group_ids       = [
        "sg-0123456789abcdef0",
    ]
    tags                         = {
        "Name" = "manual-web-server"
    }
}
```

##### Step 4: Update `main.tf` to Match the Live Attributes
Copy the non-computed attributes from the state output into `main.tf`:

```hcl
resource "aws_instance" "web_server" {
  ami                    = "ami-0c55b159cbfafe1f0"
  instance_type          = "t3.micro"
  key_name               = "my-ssh-key"
  subnet_id              = "subnet-0a1b2c3d4e5f6g7h8"
  vpc_security_group_ids = ["sg-0123456789abcdef0"]

  tags = {
    Name = "manual-web-server"
  }
}
```

##### Step 5: Verify with `terraform plan` (Zero Drift Check)
Run `terraform plan` to confirm that Terraform detects **no diff**:

```bash
terraform plan
```

Output:
```text
No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.
```
If `terraform plan` shows any diff (such as `~ update in-place` or `# forces replacement`), adjust your `main.tf` attributes until the output reports **No changes**.

---

### 🧩 Common Cloud Resource IDs for `terraform import`

Every cloud resource requires a specific identifier format when importing. You can find the exact format at the bottom of each resource's page in the official Terraform Registry documentation:

| Resource Type | Resource Address | Import ID Format | Example CLI Command |
| :--- | :--- | :--- | :--- |
| **EC2 Instance** | `aws_instance.<name>` | `instance-id` | `terraform import aws_instance.web i-0123456789abcdef0` |
| **S3 Bucket** | `aws_s3_bucket.<name>` | `bucket-name` | `terraform import aws_s3_bucket.data my-company-bucket` |
| **Security Group** | `aws_security_group.<name>` | `security-group-id` | `terraform import aws_security_group.sg sg-0123456789abcdef0` |
| **VPC** | `aws_vpc.<name>` | `vpc-id` | `terraform import aws_vpc.main vpc-0123456789abcdef0` |
| **Subnet** | `aws_subnet.<name>` | `subnet-id` | `terraform import aws_subnet.sub subnet-0123456789abcdef0` |
| **IAM Role** | `aws_iam_role.<name>` | `role-name` | `terraform import aws_iam_role.app app-execution-role` |
| **RDS Instance** | `aws_db_instance.<name>` | `db-instance-identifier` | `terraform import aws_db_instance.db prod-mysql-db` |

---

### ⚠️ Critical Pitfalls & Best Practices

1. **Beware of "Forces Replacement"**:
   - Certain attributes (such as changing `ami`, `availability_zone`, or `subnet_id`) cannot be modified in-place by AWS. If your HCL differs from the imported state on these attributes, Terraform will attempt to **destroy and re-create** your running server!
   - Always run `terraform plan` before `terraform apply` to ensure no unexpected resource recreation occurs.

2. **Importing Connected Dependencies**:
   - An EC2 instance typically depends on Security Groups, Elastic IPs, IAM Instance Profiles, and Key Pairs.
   - For complete IaC management, import these related resources as well, or reference them as data sources (`data "aws_security_group" "..."`).

3. **Importing into Modules**:
   - When importing a resource inside a child module using the CLI:
     ```bash
     terraform import module.compute.aws_instance.web i-0123456789abcdef0
     ```
   - In modern declarative `import` blocks:
     ```hcl
     import {
       to = module.compute.aws_instance.web
       id = "i-0123456789abcdef0"
     }
     ```

4. **How to Unmanage a Resource Without Deleting It (`terraform state rm`)**:
   - If you ever need to remove a resource from Terraform's management without destroying the real cloud resource:
     ```bash
     terraform state rm aws_instance.web_server
     ```
   - This removes the instance from `terraform.tfstate`. The physical EC2 instance continues running unharmed in AWS. You can then safely delete the resource block from your `.tf` files.

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

### ❓ What are Terraform Workspaces?

By default, every Terraform working directory initializes with a single, default workspace named **`default`**.

A **Workspace** allows a single Terraform configuration codebase (the exact same `.tf` files) to maintain **multiple, completely isolated state files (`terraform.tfstate`)**. This enables you to deploy identical or scaled infrastructure across different environments (e.g., `dev`, `staging`, `prod`) from a single codebase without duplicating your `.tf` files.

```
                  ┌──────────────────────────────────────────────┐
                  │          Single Terraform Codebase           │
                  │   (main.tf, variables.tf, outputs.tf)        │
                  └──────────────────────┬───────────────────────┘
                                         │
                 ┌───────────────────────┼───────────────────────┐
                 ▼                       ▼                       ▼
       ┌───────────────────┐   ┌───────────────────┐   ┌───────────────────┐
       │  Workspace: dev   │   │Workspace: staging │   │  Workspace: prod  │
       └─────────┬─────────┘   └─────────┬─────────┘   └─────────┬─────────┘
                 │                       │                       │
                 ▼                       ▼                       ▼
       ┌───────────────────┐   ┌───────────────────┐   ┌───────────────────┐
       │   dev Statefile   │   │ staging Statefile │   │  prod Statefile   │
       │ (env:/dev/tfstate)│   │(env:/stag/tfstate)│   │(env:/prod/tfstate)│
       └───────────────────┘   └───────────────────┘   └───────────────────┘
```

#### Where is Workspace State Stored?
- **Local Backend**: Stored under directory `terraform.tfstate.d/<workspace-name>/terraform.tfstate`.
- **Remote Backend (AWS S3)**: Automatically stored under an environment prefix:
  `s3://<bucket-name>/env:/<workspace-name>/<key-path>`.
  - For `dev`: `s3://my-state-bucket/env:/dev/prod/terraform.tfstate`
  - For `prod`: `s3://my-state-bucket/env:/prod/prod/terraform.tfstate`
  *(The `default` workspace remains at the root key path without the `env:/` prefix).*

---

### ⌨️ Workspace CLI Commands & Lifecycle

```bash
# 1. List all available workspaces (* marks the active one)
terraform workspace list

# 2. Create a new workspace and switch to it immediately
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# 3. Switch between existing workspaces
terraform workspace select dev
terraform workspace select prod

# 4. Display the currently active workspace name
terraform workspace show

# 5. Delete an unused workspace (Must switch away first; cannot delete active or 'default')
terraform workspace select default
terraform workspace delete dev
```

---

### ⚙️ Configuring Dynamic Infrastructure with `${terraform.workspace}`

Terraform exposes the active workspace name as a built-in variable: **`terraform.workspace`**. You can dynamically adjust names, sizing, instance counts, and costs based on the active workspace.

#### Pattern 1: Lookup Maps for Environment Sizing (Cleanest Pattern)
Instead of messy nested ternary conditions, use local maps with `lookup()`:

```hcl
locals {
  # Instance type per environment
  instance_types = {
    dev     = "t3.micro"
    staging = "t3.small"
    prod    = "m5.large"
  }

  # Node count per environment
  instance_counts = {
    dev     = 1
    staging = 2
    prod    = 5
  }

  # Selected values with fallback default
  selected_instance_type  = lookup(local.instance_types, terraform.workspace, "t3.micro")
  selected_instance_count = lookup(local.instance_counts, terraform.workspace, 1)
}

resource "aws_instance" "web" {
  count         = local.selected_instance_count
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = local.selected_instance_type

  tags = {
    Name        = "${terraform.workspace}-web-${count.index}"
    Environment = terraform.workspace
    ManagedBy   = "Terraform"
  }
}
```

#### Pattern 2: Workspace-Specific Variable Files (`.tfvars`)
Keep configuration variables separated by creating environment-specific variable files:

```
├── main.tf
├── variables.tf
├── outputs.tf
├── dev.tfvars       # Specific to dev (e.g., db_size = "db.t3.micro")
├── staging.tfvars   # Specific to staging (e.g., db_size = "db.t3.small")
└── prod.tfvars      # Specific to prod (e.g., db_size = "db.m5.large", multi_az = true)
```

Apply using shell interpolation:
```bash
# Switch workspace and pass matching .tfvars:
terraform workspace select dev
terraform apply -var-file="dev.tfvars"

# Or dynamically pass in scripts/CI:
terraform apply -var-file="$(terraform workspace show).tfvars"
```

---

### 🌿 Git Branches & Terraform Workspaces: Multi-Environment Setup

A standard DevOps practice is aligning **Git branch strategy** with **Terraform workspaces** to drive infrastructure deployments automatically through GitOps.

```
Git Branch:               dev                   staging                  main
                           │                       │                       │
                           ▼                       ▼                       ▼
CI/CD Pipeline:       Triggers dev job       Triggers staging job     Triggers prod job
                           │                       │                       │
Terraform Workspace:   select "dev"           select "staging"        select "prod"
                           │                       │                       │
Applied Var File:     -var-file=dev.tfvars   -var-file=staging.tfvars -var-file=prod.tfvars
                           │                       │                       │
Deployment Target:    Low-cost Sandbox         Pre-prod Staging       High-Availability
                      (t3.micro, Single-AZ)   (t3.small, Multi-AZ)    (m5.large, Multi-AZ)
```

---

#### 1. Branch-to-Workspace Mapping Strategy

| Git Branch | Terraform Workspace | Var File | Purpose | Safeguards |
| :--- | :--- | :--- | :--- | :--- |
| **`dev`** | `dev` | `dev.tfvars` | Daily feature testing, developers integrate code | Auto-apply on merge |
| **`staging`** | `staging` | `staging.tfvars` | Pre-production testing, QA validation, UAT | Auto-apply on merge |
| **`main` / `master`** | `prod` | `prod.tfvars` | Live customer-facing production infrastructure | Manual Approval Gate required |
| **`feat/*`** *(optional)* | `feat-<pr_id>` | `dev.tfvars` | **Ephemeral Environments**: Created for a PR and destroyed on merge | Auto-destroy on PR close |

---

#### 2. Production CI/CD Pipeline (GitHub Actions)

Here is a complete, real-world GitHub Actions workflow (`.github/workflows/terraform-multi-env.yml`) that automatically detects the branch, selects or creates the corresponding Terraform workspace, and applies the right environment variables:

```yaml
name: "Terraform Multi-Environment Deployment"

on:
  push:
    branches:
      - dev
      - staging
      - main
  pull_request:
    branches:
      - dev
      - staging
      - main

jobs:
  terraform:
    name: "Terraform Run"
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.7.0

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      # Map Git branch to Terraform Workspace:
      # 'main' maps to 'prod', while 'dev' and 'staging' map directly
      - name: Determine Environment & Workspace
        id: env
        run: |
          if [ "${{ github.ref_name }}" == "main" ]; then
            echo "WORKSPACE=prod" >> $GITHUB_OUTPUT
            echo "ENV_NAME=production" >> $GITHUB_OUTPUT
          elif [ "${{ github.ref_name }}" == "staging" ]; then
            echo "WORKSPACE=staging" >> $GITHUB_OUTPUT
            echo "ENV_NAME=staging" >> $GITHUB_OUTPUT
          else
            echo "WORKSPACE=dev" >> $GITHUB_OUTPUT
            echo "ENV_NAME=development" >> $GITHUB_OUTPUT
          fi

      - name: Terraform Init
        run: terraform init

      # Select workspace if it exists, or create it dynamically
      - name: Select or Create Workspace
        run: |
          WORKSPACE="${{ steps.env.outputs.WORKSPACE }}"
          echo "Switching to workspace: $WORKSPACE"
          terraform workspace select $WORKSPACE || terraform workspace new $WORKSPACE

      # Run Plan for Pull Requests
      - name: Terraform Plan
        if: github.event_name == 'pull_request'
        run: |
          WORKSPACE="${{ steps.env.outputs.WORKSPACE }}"
          terraform plan -var-file="${WORKSPACE}.tfvars"

      # Run Apply on Push (with GitHub Environment approval for prod)
      - name: Terraform Apply
        if: github.event_name == 'push'
        run: |
          WORKSPACE="${{ steps.env.outputs.WORKSPACE }}"
          terraform apply -auto-approve -var-file="${WORKSPACE}.tfvars"
```

---

#### 3. Ephemeral / Preview Environments (PR-Based Workspaces)

Workspaces are exceptionally well-suited for **ephemeral preview environments** (spin up temporary infrastructure for a PR, test it, and tear it down upon merge):

1. **On PR Open**:
   ```bash
   # CI dynamically creates a dedicated workspace for the PR:
   terraform workspace new pr-${{ github.event.number }}
   terraform apply -auto-approve -var-file="dev.tfvars"
   ```
2. **On PR Close / Merge**:
   ```bash
   # CI tears down resources and cleans up the workspace:
   terraform workspace select pr-${{ github.event.number }}
   terraform destroy -auto-approve -var-file="dev.tfvars"
   terraform workspace select default
   terraform workspace delete pr-${{ github.event.number }}
   ```

---

### ⚖️ Architectural Comparison: Workspaces vs. Separate Directories

While Workspaces are powerful, industry-leading teams carefully evaluate when to use **Workspaces** versus **Directory-Based Isolation**:

| Consideration | Workspaces Pattern (`terraform workspace`) | Directory Isolation Pattern (`/dev`, `/staging`, `/prod`) |
| :--- | :--- | :--- |
| **Code Duplication** | **Zero (100% DRY)**: Same `.tf` files for all envs | Requires modules or symlinks across directories |
| **State Isolation** | Isolated keys in same backend (`env:/dev/...`) | Completely separate backend buckets & keys |
| **Account / IAM Isolation** | Harder: typically shares the same cloud account | **Easy**: Dev AWS Account vs. Prod AWS Account |
| **Blast Radius** | Higher: A bug in code or bad apply can hit prod | **Minimal**: Changes in `dev/` cannot affect `prod/` |
| **Provider Versions** | All envs must use identical provider versions | Can test new provider versions in `dev/` first |
| **Human Error Risk** | High: Forgetting to switch workspace applies to wrong env | Low: You are explicitly in the `environments/prod/` path |
| **Best Use Case** | Ephemeral environments, feature branches, simple multi-tier envs | **Enterprise Production**: Multi-account AWS architecture |

> [!TIP]
> **The Recommended Industry Hybrid Strategy**:
> - Use **Separate Directories / Cloud Accounts** for major long-lived environments:
>   - `AWS Account 1 (Non-Prod)`: Contains `/environments/dev` and `/environments/staging`.
>   - `AWS Account 2 (Prod)`: Contains `/environments/prod` with strict IAM boundaries.
> - Use **Workspaces** within the non-prod account to spin up fast, ephemeral developer feature branches (`feat-auth`, `pr-42`).

---

### 🛡️ Production Safeguards for Workspaces

To prevent engineers from accidentally modifying or destroying production when switching workspaces:

1. **Protect Sensitive Resources with `lifecycle`**:
   ```hcl
   resource "aws_db_instance" "database" {
     # ...
     lifecycle {
       # Prevents accidental terraform destroy in production
       prevent_destroy = true
     }
   }
   ```

2. **Add Workspace Guardrail in Code**:
   Prevent running `apply` in the `default` workspace:
   ```hcl
   # Fail immediately if someone accidentally runs on 'default':
   check "workspace_check" {
     assert {
       condition     = terraform.workspace != "default"
       error_message = "Deployment to 'default' workspace is strictly forbidden! Use 'dev', 'staging', or 'prod'."
     }
   }
   ```

3. **Shell Prompt Display**:
   Configure your terminal (e.g. Starship or Powerlevel10k) to show the active Terraform workspace in your prompt:
   ```bash
   # Add to ~/.bashrc or ~/.zshrc:
   export PS1='[\u@\h \W $(terraform workspace show 2>/dev/null)]\$ '
   ```

---

## 🧩 Terraform Modules (Reusable Infrastructure Blocks)

### ❓ What is a Terraform Module?

A **Terraform Module** is a container for multiple resources that are used together. In programming terms, if resources are individual statements, a **module is like a function or class**:

- **Input Arguments** $\longrightarrow$ **`variables.tf`** (Parameters passed into the module)
- **Function Body / Execution Logic** $\longrightarrow$ **`main.tf`** (Resources created inside the module)
- **Return Values** $\longrightarrow$ **`outputs.tf`** (Values calculated and exported back to the caller)

```
                       CALLER (Root Module)
                       ┌────────────────────────────────────────────────────────┐
                       │ module "web_cluster" {                                 │
                       │   source        = "./modules/aws-web-server"           │
                       │   instance_type = "t3.micro"  ──┐ (Input Variable)     │
                       │   environment   = "production"──┤                      │
                       │ }                               │                      │
                       └─────────────────────────────────┼──────────────────────┘
                                                         │
                                                         ▼
                                          CHILD MODULE (aws-web-server)
                                          ┌─────────────────────────────────────┐
                                          │ variables.tf:                       │
                                          │   variable "instance_type" {}       │
                                          │                                     │
                                          │ main.tf:                            │
                                          │   resource "aws_instance" "web" {   │
                                          │     instance_type = var.instance_type
                                          │   }                                 │
                                          │   resource "aws_security_group" ... │
                                          │                                     │
                                          │ outputs.tf:                         │
                                          │   output "public_ip" { ... } ───────┼┐
                                          └─────────────────────────────────────┘│
                                                                                 │
                       CALLER RECEIVES OUTPUT (module.web_cluster.public_ip) ◄───┘
```

#### Why Use Modules?
1. **DRY (Don't Repeat Yourself)**: Avoid copying and pasting 50 lines of VPC or EC2 boilerplate across multiple projects or environments.
2. **Standardization & Compliance**: Enforce corporate security policies, default tagging, encryption, and logging standards in one centralized place.
3. **Encapsulation**: Hide internal complexity (such as route tables, subnets, NAT gateways) behind clean, simple input parameters.
4. **Independent Versioning**: Version your infrastructure modules using Git tags (`v1.0.0`, `v2.0.0`), allowing safe progressive rollouts across environments.

---

### 📂 Standard Module Structure & Anatomy

HashiCorp defines a standard directory layout for reusable Terraform modules:

```
modules/aws-web-server/
├── README.md        # Documentation: inputs, outputs, prerequisites, usage examples
├── main.tf          # Core infrastructure resources (EC2, Security Groups, IAM)
├── variables.tf     # Input variables (with description, type, default, validation)
├── outputs.tf       # Exported attributes accessible to parent modules
├── versions.tf      # Required Terraform version & provider constraints
└── examples/        # Working example configurations for consumers
    └── basic/
        └── main.tf
```

---

### 🛠️ Hands-On: Building a Custom Reusable EC2 Web Server Module

Let's build a real-world child module that provisions an **EC2 instance**, creates an **associated Security Group**, and optionally attaches an **Elastic IP**.

#### 1. Define Input Variables (`modules/aws-web-server/variables.tf`)

```hcl
variable "instance_name" {
  description = "Name tag for the EC2 instance"
  type        = string
}

variable "ami_id" {
  description = "AMI ID to launch the instance with"
  type        = string
}

variable "instance_type" {
  description = "EC2 instance sizing"
  type        = string
  default     = "t3.micro"
}

variable "vpc_id" {
  description = "VPC ID where the security group will be created"
  type        = string
}

variable "subnet_id" {
  description = "Subnet ID where the instance will reside"
  type        = string
}

variable "server_port" {
  description = "Port to open for web traffic (e.g. 80 or 8080)"
  type        = number
  default     = 80
}

variable "enable_elastic_ip" {
  description = "Whether to allocate and associate a static Elastic IP"
  type        = bool
  default     = false
}

variable "tags" {
  description = "Additional tags to apply to all module resources"
  type        = map(string)
  default     = {}
}
```

#### 2. Define Core Resources (`modules/aws-web-server/main.tf`)

```hcl
# 1. Security Group dedicated to this web server
resource "aws_security_group" "web_sg" {
  name        = "${var.instance_name}-sg"
  description = "Security group for ${var.instance_name}"
  vpc_id      = var.vpc_id

  ingress {
    description = "Allow inbound HTTP/custom web traffic"
    from_port   = var.server_port
    to_port     = var.server_port
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "Allow all outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(var.tags, {
    Name = "${var.instance_name}-sg"
  })
}

# 2. EC2 Instance
resource "aws_instance" "server" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello from ${var.instance_name}" > index.html
              python3 -m http.server ${var.server_port} &
              EOF

  tags = merge(var.tags, {
    Name = var.instance_name
  })
}

# 3. Optional Elastic IP (Conditional Resource)
resource "aws_eip" "server_eip" {
  count    = var.enable_elastic_ip ? 1 : 0
  instance = aws_instance.server.id
  domain   = "vpc"

  tags = merge(var.tags, {
    Name = "${var.instance_name}-eip"
  })
}
```

#### 3. Define Return Values (`modules/aws-web-server/outputs.tf`)

```hcl
output "instance_id" {
  description = "The ID of the provisioned EC2 instance"
  value       = aws_instance.server.id
}

output "security_group_id" {
  description = "The ID of the created security group"
  value       = aws_security_group.web_sg.id
}

output "public_ip" {
  description = "The public IP address (EIP if enabled, otherwise EC2 public IP)"
  value       = var.enable_elastic_ip ? aws_eip.server_eip[0].public_ip : aws_instance.server.public_ip
}
```

#### 4. Define Provider Constraints (`modules/aws-web-server/versions.tf`)

> [!NOTE]
> Child modules should declare **`required_providers`** to define compatibility constraints, but should **NOT** define `provider "aws" { region = ... }` configuration blocks. Providers should be configured in the Root Module.

```hcl
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 4.0, < 6.0"
    }
  }
}
```

---

### 📞 Calling the Module from the Root Module (`main.tf`)

In your root working directory, invoke the child module using a `module` block:

```hcl
# root/main.tf
provider "aws" {
  region = "us-east-1"
}

# Invoke the custom child module:
module "frontend_web" {
  source = "./modules/aws-web-server"

  instance_name     = "production-frontend"
  ami_id            = "ami-0c55b159cbfafe1f0"
  instance_type     = "t3.small"
  vpc_id            = "vpc-0123456789abcdef0"
  subnet_id         = "subnet-0123456789abcdef0"
  server_port       = 80
  enable_elastic_ip = true

  tags = {
    Environment = "production"
    Team        = "Frontend-DevOps"
  }
}

# Accessing module output values:
output "frontend_url" {
  description = "URL to access the frontend web server"
  value       = "http://${module.frontend_web.public_ip}"
}
```

---

### 🌐 Module Sources (Where Can You Load Modules From?)

The `source` argument tells Terraform where to fetch the module code.

#### 1. Local File Path
Used for modules within the same repository:
```hcl
module "vpc" {
  source = "./modules/vpc"    # Relative path
  # or source = "../shared/modules/vpc"
}
```

#### 2. Official Terraform Registry
Community-vetted, public modules maintained by cloud vendors and the community:
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0" # Always pin the version!

  name = "production-vpc"
  cidr = "10.0.0.0/16"
  azs  = ["us-east-1a", "us-east-1b"]
}
```

#### 3. Git Repositories (Private & Public)
Load modules directly from GitHub, GitLab, or Bitbucket.

- **HTTPS**:
  ```hcl
  source = "git::https://github.com/my-org/terraform-aws-ec2.git"
  ```
- **SSH (Recommended for Private Repos)**:
  ```hcl
  source = "git::git@github.com:my-org/terraform-aws-ec2.git"
  ```
- **Pinning Specific Git Branches, Tags, or Commits via `?ref=`**:
  ```hcl
  # Pin to a Git Tag (Recommended for Production stability)
  source = "git::git@github.com:my-org/terraform-aws-ec2.git?ref=v2.1.0"

  # Or pin to a specific branch:
  source = "git::git@github.com:my-org/terraform-aws-ec2.git?ref=feature-branch"
  ```
- **Monorepo Subdirectories (Double Slash `//`)**:
  If multiple modules live in one repository, use `//` to target a subfolder:
  ```hcl
  source = "git::git@github.com:my-org/terraform-monorepo.git//modules/networking/vpc?ref=v1.4.0"
  ```

#### 4. Amazon S3 Bucket
Store pre-packaged zip archives in private S3 buckets:
```hcl
source = "s3::https://s3-us-east-1.amazonaws.com/company-tf-modules/vpc-module.zip"
```

---

### 🎛️ Meta-Arguments in Module Blocks

Just like standard `resource` blocks, `module` blocks support powerful meta-arguments:

#### 1. `count` with Modules (Multi-Instance Deployment)
Deploy multiple instances of a module using an index:

```hcl
module "microservices" {
  count  = 3
  source = "./modules/microservice"

  service_name = "service-${count.index + 1}"
}

# Referencing outputs from count modules:
output "all_service_ips" {
  value = module.microservices[*].public_ip
}
```

#### 2. `for_each` with Modules (Key-Based Deployment — Preferred)
Loop over a map or set to provision unique module stacks:

```hcl
locals {
  microservices = {
    auth    = { port = 8081, size = "t3.micro" }
    billing = { port = 8082, size = "t3.small" }
    orders  = { port = 8083, size = "t3.medium" }
  }
}

module "services" {
  for_each = local.microservices
  source   = "./modules/aws-web-server"

  instance_name = "svc-${each.key}"
  instance_type = each.value.size
  server_port   = each.value.port
  ami_id        = "ami-0c55b159cbfafe1f0"
  vpc_id        = "vpc-0123456789abcdef0"
  subnet_id     = "subnet-0123456789abcdef0"
}

# Accessing output of a specific module instance:
output "billing_ip" {
  value = module.services["billing"].public_ip
}
```

#### 3. `providers` (Passing Provider Configurations / Multi-Region)
Pass specific provider instances (e.g. disaster recovery regions) to child modules:

```hcl
provider "aws" {
  alias  = "dr_west"
  region = "us-west-2"
}

module "dr_backup_server" {
  source = "./modules/aws-web-server"

  providers = {
    aws = aws.dr_west # The child module's AWS resources will deploy to us-west-2
  }

  instance_name = "dr-standby"
  ami_id        = "ami-west-123456"
  vpc_id        = "vpc-west-0123456"
  subnet_id     = "subnet-west-0123456"
}
```

#### 4. `depends_on` with Modules
Force an entire module to wait until another resource or module has fully completed:

```hcl
module "eks_cluster" {
  source = "./modules/eks"
  # ...
  depends_on = [module.vpc] # Ensures VPC and subnets are fully active before EKS starts
}
```

---

### 📦 Updating & Downloading Modules (`terraform init` / `get`)

When adding, changing, or updating module sources or versions:

```bash
# Downloads module code referenced in configuration into .terraform/modules/
terraform init

# Update already downloaded modules to latest allowed matching versions
terraform init -upgrade

# Or download modules without reinitializing backend or provider plugins:
terraform get -update
```

---

### 💡 Module Design Best Practices & Anti-Patterns

| Practice | Recommendation | Why? |
| :--- | :--- | :--- |
| **Keep Modules Flat** | Avoid nesting deeper than 2 levels (`root` $\rightarrow$ `module`) | Deeply nested modules ("Spaghetti modules") are fragile, painful to debug, and tightly coupled. |
| **Pin Versions Strictly** | Always specify `version = "x.y.z"` or `?ref=v1.2.0` | Prevents upstream breaking changes from crashing production during CI/CD applies. |
| **No Hardcoded Providers** | Never declare `provider "aws" { ... }` inside child modules | Blocks consumer flexibility, multi-region aliases, and dynamic credentials. |
| **Document with README** | Use tools like `terraform-docs` to auto-generate markdown tables | Makes modules easily consumable by other development teams. |
| **Provide Sensible Defaults** | Set safe defaults for non-mandatory variables | Reduces consumer boilerplate while allowing customization when required. |
| **Validate Inputs** | Add `validation { condition = ... }` blocks to input variables | Catches misconfigurations early during `terraform plan` rather than failing mid-apply. |

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
    branches: ["main"]
  pull_request:
    branches: ["main"]

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
