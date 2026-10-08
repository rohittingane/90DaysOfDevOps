# Day 67 – TerraWeek Capstone: Multi-Environment Infrastructure with Modules and Workspaces

> **Goal of today:** Build one Terraform project that creates **three separate environments** (**dev, staging, prod**) on AWS from the **same code**.
> We use **modules** (reusable building blocks), **workspaces** (separate state per environment) and **tfvars files** (different settings per environment). At the end we verify everything in the AWS Console and destroy it all.

This guide is written for **beginners**. Every task has:

- **What** – what we are doing
- **Why** – why it matters in real projects
- **How** – exact commands and code (copy and paste)
- **Expected output** – what you should see
- **Screenshots** – real output from my run

---

## Table of Contents

1. [What we build](#1-what-we-build)
2. [Key words explained](#2-key-words-explained-in-simple-english)
3. [Before you start](#3-before-you-start)
4. [Task 1 – Learn workspaces](#task-1--learn-terraform-workspaces)
5. [Task 2 – Project structure](#task-2--create-the-project-structure)
6. [Task 3 – Write the 3 modules](#task-3--write-the-three-modules)
7. [Task 4 – Write the root files](#task-4--write-the-root-files)
8. [Task 5 – Deploy dev, staging and prod](#task-5--deploy-dev-staging-and-prod)
9. [Task 6 – Best practices guide](#task-6--best-practices-guide)
10. [Task 7 – Destroy and clean up](#task-7--destroy-everything-and-clean-up)
11. [Before you push to GitHub](#before-you-push-to-github)
12. [Summary](#summary)

---

## 1. What we build

Each environment gets the **same 7 resources**:

| # | Resource | Made by module |
|---|---|---|
| 1 | VPC | `vpc` |
| 2 | Public subnet | `vpc` |
| 3 | Internet gateway | `vpc` |
| 4 | Route table | `vpc` |
| 5 | Route table association | `vpc` |
| 6 | Security group | `security-group` |
| 7 | EC2 instance | `ec2` |

3 environments × 7 resources = **21 resources** in total.

```
                 ONE CODE BASE (modules + main.tf)
                              │
        ┌─────────────────────┼─────────────────────┐
   workspace: dev       workspace: staging     workspace: prod
   + dev.tfvars         + staging.tfvars       + prod.tfvars
        │                     │                     │
   own state file        own state file        own state file
   7 resources           7 resources           7 resources
```

---

## 2. Key words explained in simple English

| Word | Simple meaning |
|---|---|
| **Module** | A folder of Terraform code you can reuse. Like a **recipe**: write once, cook many times. |
| **Root module** | The main folder where you run `terraform` commands. It *calls* the other modules. |
| **Workspace** | A named copy of the **state**. Same code, different state per workspace. |
| **State file** | Terraform's notebook of what it created. Each workspace has its own. |
| **tfvars file** | A file with values for variables (for example instance type). One file per environment. |
| **`terraform.workspace`** | A built-in value that holds the name of the current workspace. |
| **Locals** | Small calculated values inside your code (for example a name prefix). |
| **Output** | A value Terraform prints after apply (for example the server IP). |
| **Environment** | A separate copy of your app: dev (testing), staging (final check), prod (real users). |

**Easy example:** A school has one **blueprint** for a classroom (module). Dev, staging and prod are three classrooms built from that blueprint. Each classroom has its own **attendance register** (state per workspace) and its own **size** (tfvars).

---

## 3. Before you start

- AWS account and **AWS CLI** configured (`aws sts get-caller-identity` works)
- **Terraform** installed (I used `1.16.4`)
- IAM user with **EC2 and VPC** permissions
- A Linux terminal (I used an Ubuntu EC2 machine)
- Region used in this guide: `eu-north-1` (Stockholm). Use your own region if different.

Project folder: `~/terraweek-capstone`

> 💰 **Cost note:** 3 EC2 instances run at the same time. Destroy everything at the end (Task 7).

---

## Task 1 – Learn Terraform Workspaces

### What
Create three workspaces (`dev`, `staging`, `prod`), switch between them, and see that each one has its **own state**.

### Why
Without workspaces, one state file would hold dev, staging and prod together. One wrong `destroy` could delete production. With workspaces, each environment has separate state.

### How

**Step 1 – Make a small practice folder and initialise**

```bash
mkdir ~/workspace-practice && cd ~/workspace-practice
cat > main.tf <<'EOF'
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "eu-north-1"
}

output "current_workspace" {
  value = terraform.workspace
}
EOF
terraform init
```

**Step 2 – See the default workspace and create `dev`**

```bash
terraform workspace list
terraform workspace new dev
```

`terraform workspace new` also **switches** you to the new workspace.

![Init and create dev workspace](Screenshots/01-init-and-create-dev-workspace.png)

**Step 3 – Create `staging` and `prod`, then switch around**

```bash
terraform workspace new staging
terraform workspace new prod
terraform workspace list
terraform workspace select dev
terraform workspace show
```

The `*` in the list shows the workspace you are in.

![Create staging and prod and switch](Screenshots/02-create-staging-prod-and-switch.png)

**Step 4 – Read the workspace name inside code**

```bash
terraform workspace select prod
terraform apply
```

Output shows `current_workspace = "prod"`. This is how code can know which environment it runs in.

![Workspace output prod](Screenshots/03-terraform-workspace-output-prod.png)

**Step 5 – Apply in dev and staging**

```bash
terraform workspace select dev
terraform apply
terraform workspace select staging
terraform apply
```

![Apply in dev and staging](Screenshots/04-apply-dev-and-staging.png)

**Step 6 – Look at the state files**

```bash
ls -R terraform.tfstate.d
```

Each workspace has its own folder with its own `terraform.tfstate`.

```
terraform.tfstate.d/
├── dev/terraform.tfstate
├── prod/terraform.tfstate
└── staging/terraform.tfstate
```

![State files per workspace](Screenshots/05-state-files-per-workspace.png)

> ✅ **Lesson:** one folder, one code, **three separate states**.

---

## Task 2 – Create the Project Structure

### What
Create the folders and empty files for the real project.

### Why
A clean structure makes the project easy to read. Each module has the same three files: `main.tf` (resources), `variables.tf` (inputs), `outputs.tf` (results).

### How

```bash
mkdir -p ~/terraweek-capstone && cd ~/terraweek-capstone
mkdir -p modules/vpc modules/security-group modules/ec2

touch modules/vpc/{main.tf,variables.tf,outputs.tf}
touch modules/security-group/{main.tf,variables.tf,outputs.tf}
touch modules/ec2/{main.tf,variables.tf,outputs.tf}
touch providers.tf variables.tf locals.tf main.tf outputs.tf
touch dev.tfvars staging.tfvars prod.tfvars
```

![Create folders and files](Screenshots/06-create-folders-and-files.png)

**Verify the structure**

```bash
find . -type f | sort
```

Final structure:

```
terraweek-capstone/
├── modules/
│   ├── vpc/             (main.tf, variables.tf, outputs.tf)
│   ├── security-group/  (main.tf, variables.tf, outputs.tf)
│   └── ec2/             (main.tf, variables.tf, outputs.tf)
├── providers.tf
├── variables.tf
├── locals.tf
├── main.tf
├── outputs.tf
├── dev.tfvars
├── staging.tfvars
└── prod.tfvars
```

![Verify project structure](Screenshots/07-project-structure-verify.png)

---

## Task 3 – Write the Three Modules

### What
Write the code for the **VPC**, **Security Group** and **EC2** modules.

### Why
A module is written once and used for all environments. Dev, staging and prod all call the **same module** with **different inputs**. If you fix a bug in the module, all environments get the fix.

How a module works:

```
 INPUTS (variables.tf)  ─▶  RESOURCES (main.tf)  ─▶  OUTPUTS (outputs.tf)
 "what do you want?"        "build it"                "here is the ID"
```

### 3.1 VPC module

**`modules/vpc/variables.tf`**

```hcl
variable "name_prefix" {
  type        = string
  description = "Prefix for all names, for example terraweek-dev"
}

variable "vpc_cidr" {
  type        = string
  description = "IP range of the VPC"
}

variable "public_subnet_cidr" {
  type        = string
  description = "IP range of the public subnet"
}

variable "tags" {
  type    = map(string)
  default = {}
}
```

![VPC module variables](Screenshots/08-vpc-module-variables.png)

**`modules/vpc/main.tf`**

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}

resource "aws_vpc" "this" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true

  tags = merge(var.tags, { Name = "${var.name_prefix}-vpc" })
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.this.id
  cidr_block              = var.public_subnet_cidr
  availability_zone       = data.aws_availability_zones.available.names[0]
  map_public_ip_on_launch = true

  tags = merge(var.tags, { Name = "${var.name_prefix}-public-subnet" })
}

resource "aws_internet_gateway" "this" {
  vpc_id = aws_vpc.this.id

  tags = merge(var.tags, { Name = "${var.name_prefix}-igw" })
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.this.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.this.id
  }

  tags = merge(var.tags, { Name = "${var.name_prefix}-public-rt" })
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```

![VPC module main part 1](Screenshots/08-vpc-module-main-1.png)

**Logic in simple words**
- The VPC is your private network in AWS.
- The subnet is a smaller part of it. `map_public_ip_on_launch = true` gives servers a public IP.
- The internet gateway is the **door to the internet**.
- The route table says "send all internet traffic (`0.0.0.0/0`) through the door".
- The association connects the route table to the subnet.

**`modules/vpc/outputs.tf`**

```hcl
output "vpc_id" {
  value = aws_vpc.this.id
}

output "public_subnet_id" {
  value = aws_subnet.public.id
}
```

![VPC module main part 2 and outputs](Screenshots/09-vpc-module-main-2-outputs.png)

**Test the module**

```bash
cd ~/terraweek-capstone/modules/vpc
terraform init
terraform validate
cd ~/terraweek-capstone
```

Expected: `Success! The configuration is valid.`

![VPC module validate](Screenshots/10-vpc-module-validate.png)

### 3.2 Security Group module

**`modules/security-group/variables.tf`**

```hcl
variable "name_prefix" {
  type = string
}

variable "vpc_id" {
  type        = string
  description = "VPC where the security group is created"
}

variable "ingress_ports" {
  type        = list(number)
  description = "Ports to allow from the internet"
  default     = [80]
}

variable "tags" {
  type    = map(string)
  default = {}
}
```

**`modules/security-group/main.tf`**

```hcl
resource "aws_security_group" "this" {
  name        = "${var.name_prefix}-sg"
  description = "Security group for ${var.name_prefix}"
  vpc_id      = var.vpc_id

  dynamic "ingress" {
    for_each = var.ingress_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(var.tags, { Name = "${var.name_prefix}-sg" })
}
```

![SG module variables and main part 1](Screenshots/11-sg-module-variables-main-1.png)

**Logic in simple words**
- A security group is a **firewall** for the server.
- `dynamic "ingress"` makes **one rule per port** from the list. Dev can open `[80, 22]`, prod can open `[80, 443]`, with the same code.
- `egress` allows the server to go out to the internet (updates, downloads).

**`modules/security-group/outputs.tf`**

```hcl
output "security_group_id" {
  value = aws_security_group.this.id
}
```

![SG module main part 2 and outputs](Screenshots/12-sg-module-main-2-outputs.png)

**Test**

```bash
cd ~/terraweek-capstone/modules/security-group
terraform init
terraform validate
cd ~/terraweek-capstone
```

![SG module validate](Screenshots/13-sg-module-validate.png)

### 3.3 EC2 module

**`modules/ec2/variables.tf`**

```hcl
variable "name_prefix" {
  type = string
}

variable "instance_type" {
  type        = string
  description = "Size of the server, for example t3.micro"
}

variable "subnet_id" {
  type = string
}

variable "security_group_ids" {
  type = list(string)
}

variable "tags" {
  type    = map(string)
  default = {}
}
```

![EC2 module variables](Screenshots/14-ec2-module-variables.png)

**`modules/ec2/main.tf`**

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_instance" "this" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids

  tags = merge(var.tags, { Name = "${var.name_prefix}-server" })
}
```

**Logic in simple words**
- The `data` block **finds** the latest Amazon Linux image (AMI). It does not create anything.
- The instance uses the subnet and security group that **other modules** created. These IDs are passed in from the root `main.tf`.

**`modules/ec2/outputs.tf`**

```hcl
output "instance_id" {
  value = aws_instance.this.id
}

output "public_ip" {
  value = aws_instance.this.public_ip
}
```

![EC2 module main and outputs](Screenshots/15-ec2-module-main-outputs.png)

**Test**

```bash
cd ~/terraweek-capstone/modules/ec2
terraform init
terraform validate
cd ~/terraweek-capstone
```

![EC2 module validate](Screenshots/16-ec2-module-validate.png)

---

## Task 4 – Write the Root Files

### What
Write the root files that **call the modules** and set the values for each environment.

### Why
The modules only describe *how* to build. The root module decides *what* to build and *with which values*.

### How

**Step 1 – `providers.tf`**

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.region
}
```

**Step 2 – `variables.tf`**

```hcl
variable "region" {
  type    = string
  default = "eu-north-1"
}

variable "project_name" {
  type    = string
  default = "terraweek"
}

variable "environment" {
  type        = string
  description = "dev, staging or prod"
}

variable "vpc_cidr" {
  type = string
}

variable "public_subnet_cidr" {
  type = string
}

variable "instance_type" {
  type = string
}

variable "ingress_ports" {
  type    = list(number)
  default = [80]
}
```

**Step 3 – `locals.tf`**

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"

  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
    Workspace   = terraform.workspace
  }
}
```

**Logic:** `name_prefix` becomes `terraweek-dev`, `terraweek-staging` or `terraweek-prod`. Every resource name starts with it, so you can find them in the Console by searching `terraweek`.

![Root locals and variables](Screenshots/17-root-locals-variables.png)

**Step 4 – The three tfvars files**

**`dev.tfvars`**

```hcl
environment        = "dev"
vpc_cidr           = "10.0.0.0/16"
public_subnet_cidr = "10.0.1.0/24"
instance_type      = "t3.micro"
ingress_ports      = [80, 22]
```

**`staging.tfvars`**

```hcl
environment        = "staging"
vpc_cidr           = "10.1.0.0/16"
public_subnet_cidr = "10.1.1.0/24"
instance_type      = "t3.micro"
ingress_ports      = [80]
```

**`prod.tfvars`**

```hcl
environment        = "prod"
vpc_cidr           = "10.2.0.0/16"
public_subnet_cidr = "10.2.1.0/24"
instance_type      = "t3.micro"
ingress_ports      = [80, 443]
```

> Each environment has a **different IP range** (`10.0`, `10.1`, `10.2`). This avoids clashes if you ever connect the networks. Prod would normally use a bigger instance type. I kept `t3.micro` to save cost.

![Root providers and dev.tfvars](Screenshots/18-root-providers-dev-tfvars.png)

![Staging and prod tfvars and ls](Screenshots/19-root-staging-prod-tfvars-ls.png)

**Step 5 – `main.tf` (calls the modules)**

```hcl
module "vpc" {
  source = "./modules/vpc"

  name_prefix        = local.name_prefix
  vpc_cidr           = var.vpc_cidr
  public_subnet_cidr = var.public_subnet_cidr
  tags               = local.common_tags
}

module "security_group" {
  source = "./modules/security-group"

  name_prefix   = local.name_prefix
  vpc_id        = module.vpc.vpc_id
  ingress_ports = var.ingress_ports
  tags          = local.common_tags
}

module "ec2" {
  source = "./modules/ec2"

  name_prefix        = local.name_prefix
  instance_type      = var.instance_type
  subnet_id          = module.vpc.public_subnet_id
  security_group_ids = [module.security_group.security_group_id]
  tags               = local.common_tags
}
```

**Logic in simple words** (data flows from one module to the next):

```
 vpc module ──vpc_id──────────────▶ security_group module
 vpc module ──public_subnet_id────▶ ec2 module
 security_group module ──security_group_id──▶ ec2 module
```

Terraform reads these links and builds in the right order: **VPC first, then security group, then EC2**.

![Root main.tf start](Screenshots/21-modules-outputs-main-start.png)

![Root main.tf end](Screenshots/22-root-main-tf-end.png)

**Step 6 – Initialise and validate**

```bash
terraform init
terraform validate
```

`terraform init` also downloads the AWS provider and loads the three local modules. Expected: `Success! The configuration is valid.`

![Init and validate](Screenshots/24-root-init-validate.png)

**Step 7 – Name tag fix**

While checking the plan I noticed the `Name` tag must come **last** inside `merge(...)`. In `merge`, the **later** map wins. That is why each module uses:

```hcl
tags = merge(var.tags, { Name = "${var.name_prefix}-server" })
```

If `Name` were first and `common_tags` also had a `Name`, the wrong value would win.

![Name tag fix](Screenshots/25-name-tag-fix.png)

---

## Task 5 – Deploy Dev, Staging and Prod

### What
Deploy the same code three times, once per workspace, each with its own tfvars file.

### Why
This is the real test. If the design is right, one code base creates three environments, and each has its own state, so they never affect each other.

### The golden rule

> **Always pair the workspace with the matching tfvars file.**
> `dev` workspace → `dev.tfvars`, `staging` workspace → `staging.tfvars`, `prod` workspace → `prod.tfvars`.
> Using `prod.tfvars` inside the `dev` workspace would write prod values into dev state.

### How

### 5.1 Dev

```bash
cd ~/terraweek-capstone
terraform workspace new dev
terraform workspace show
terraform plan -var-file="dev.tfvars"
```

Expected: `Plan: 7 to add, 0 to change, 0 to destroy.`

![Dev workspace and plan start](Screenshots/26-workspaces-dev-plan-start.png)

![Dev plan summary](Screenshots/27-dev-plan-summary.png)

```bash
terraform apply -var-file="dev.tfvars"
```

Type `yes`. Expected: `Apply complete! Resources: 7 added, 0 changed, 0 destroyed.`

![Dev apply start](Screenshots/28-dev-apply-start.png)

![Dev apply complete](Screenshots/29-dev-apply-complete.png)

**Add `outputs.tf`** so Terraform prints useful values after apply:

```hcl
output "environment" {
  value = var.environment
}

output "workspace" {
  value = terraform.workspace
}

output "vpc_id" {
  value = module.vpc.vpc_id
}

output "instance_id" {
  value = module.ec2.instance_id
}

output "instance_public_ip" {
  value = module.ec2.public_ip
}
```

![Root outputs.tf](Screenshots/30-root-outputs-tf.png)

```bash
terraform apply -var-file="dev.tfvars"
```

Only outputs are added (`0 added, 0 changed, 0 destroyed`). You now see the VPC ID and server IP.

![Dev outputs](Screenshots/31-dev-outputs-apply.png)

### 5.2 Staging

```bash
terraform workspace new staging
terraform plan -var-file="staging.tfvars"
```

Expected: `Plan: 7 to add`. The plan is **fresh** because staging state is empty. Dev is not touched.

![Staging plan start](Screenshots/33-staging-plan-start.png)

![Staging plan summary](Screenshots/34-staging-plan-summary.png)

```bash
terraform apply -var-file="staging.tfvars"
```

![Staging apply start](Screenshots/35-staging-apply-start.png)

![Staging apply complete](Screenshots/36-staging-apply-complete.png)

### 5.3 Prod

```bash
terraform workspace new prod
terraform plan -var-file="prod.tfvars"
```

![Prod plan start](Screenshots/37-prod-plan-start.png)

![Prod plan summary](Screenshots/38-prod-plan-summary.png)

```bash
terraform apply -var-file="prod.tfvars"
```

![Prod apply start](Screenshots/39-prod-apply-start.png)

![Prod apply complete](Screenshots/40-prod-apply-complete.png)

### 5.4 Check all workspaces together

```bash
for ws in dev staging prod; do
  terraform workspace select $ws
  echo "=== $ws ==="
  terraform output
done
```

Each workspace shows its **own** VPC ID and IP.

![All workspaces output](Screenshots/41-all-workspaces-output.png)

### 5.5 Verify in the AWS Console

Open the Console and check your region (Stockholm).

1. **VPC → Your VPCs** – three VPCs: `terraweek-dev-vpc`, `terraweek-staging-vpc`, `terraweek-prod-vpc`.
2. **EC2 → Instances** – three servers named `terraweek-<env>-server`.
3. **EC2 → Security Groups** – open the dev and prod groups and compare the inbound rules. Dev has ports 80 and 22. Prod has 80 and 443.

![Console VPCs](Screenshots/42-console-vpcs.png)

![Console EC2 instances](Screenshots/43-console-ec2.png)

![Console prod security group](Screenshots/44-console-sg-prod.png)

![Console dev security group](Screenshots/45-console-sg-dev.png)

### 5.6 Prove state isolation

```bash
terraform workspace select dev
terraform state list

terraform workspace select prod
terraform state list

ls terraform.tfstate.d/
```

Each workspace lists only **its own** 7 resources (3 modules' worth: `module.vpc...`, `module.security_group...`, `module.ec2...`). Three folders appear: `dev`, `staging`, `prod`.

![State isolation](Screenshots/46-state-isolation.png)

> ✅ **Result:** 21 resources, 3 states, 1 code base.

---

## Task 6 – Best Practices Guide

### What
Rules a team should follow when using modules and workspaces.

### Why
The capstone works, but real teams need extra safety. These rules prevent expensive mistakes.

### Best practices

**1. Keep modules small and focused**
One module = one job (VPC, security group, EC2). Small modules are easy to test and reuse.

**2. Never hard-code values in modules**
Use variables. Put environment differences in tfvars files, not in `if` logic inside code.

**3. Always pair the workspace and the tfvars file**
A wrong pair is the most common mistake with workspaces. Run `terraform workspace show` **before every apply**. In a pipeline, set both from the same variable.

**4. Use a remote backend with locking**
Local state is fine for learning. In a team, store state in **S3 with DynamoDB locking** (see Day 64). Workspaces then keep separate state files in the same bucket.

**5. Use a consistent naming pattern**
`<project>-<environment>-<resource>` (for example `terraweek-prod-vpc`). You can search, filter and find costs per environment easily.

**6. Tag every resource**
`Project`, `Environment`, `ManagedBy`. Tags help with billing reports and ownership.

**7. Pin versions**
Pin the provider (`~> 5.0`) and commit `.terraform.lock.hcl`. Everyone then gets the same provider version.

**8. Always run `plan` before `apply`**
Read the plan. Look for `-/+` (destroy and recreate) and `-` (delete).

**9. Protect production**
- Add `lifecycle { prevent_destroy = true }` on critical resources.
- Require a pull request review for prod changes.
- Use a **separate AWS account** for prod. Workspaces share one account and one code base, so a mistake in the wrong workspace can still hit prod.

**10. Never put secrets in tfvars or Git**
Use AWS Secrets Manager, SSM Parameter Store or environment variables.

**11. Do not commit state files**
State can contain secrets. Use `.gitignore`.

**12. Destroy test environments**
Dev and staging running all night cost money. Destroy them when you finish.

### Workspaces vs separate folders

| | Workspaces | Separate folders per environment |
|---|---|---|
| Code copies | One | One per environment |
| Risk of wrong environment | **Higher** (easy to forget which workspace) | Lower (you are in a clear folder) |
| Good for | Learning, small teams, similar environments | Large teams, very different environments |

---

## Task 7 – Destroy Everything and Clean Up

### What
Destroy all three environments, delete the workspaces, and confirm in the AWS Console.

### Why
Running servers cost money. Also, a clean account shows that Terraform tracked every resource properly.

### The order

1. **Destroy** each environment **with its matching tfvars file**.
2. **Switch** to the `default` workspace.
3. **Delete** the other workspaces.

> ⚠️ Terraform **cannot delete the workspace you are currently in**, and it **warns you if the workspace still has resources**. That is why we destroy first, then switch, then delete.

### How

**Step 1 – Destroy prod**

```bash
cd ~/terraweek-capstone
terraform workspace select prod
terraform workspace show
terraform destroy -var-file="prod.tfvars"
```

Type `yes`. Expected: `Destroy complete! Resources: 7 destroyed.`

![Destroy prod](Screenshots/48-destroy-prod.png)

**Step 2 – Destroy staging**

```bash
terraform workspace select staging
terraform destroy -var-file="staging.tfvars"
```

![Destroy staging](Screenshots/49-destroy-staging.png)

**Step 3 – Destroy dev**

```bash
terraform workspace select dev
terraform destroy -var-file="dev.tfvars"
```

![Destroy dev](Screenshots/50-destroy-dev.png)

**Step 4 – Confirm all states are empty**

```bash
terraform state list
```

An empty output means nothing is left. If you run `destroy` again, you see `No changes. No objects need to be destroyed.`

**Step 5 – Delete the workspaces**

```bash
terraform workspace select default
terraform workspace delete dev
terraform workspace delete staging
terraform workspace delete prod
terraform workspace list
```

Expected: only `* default` is left.

![Workspaces deleted](Screenshots/51-workspaces-deleted.png)

**Step 6 – Verify in the AWS Console**

Check the same region (Stockholm).

| Where | Search | Expected |
|---|---|---|
| **VPC → Your VPCs** | `terraweek` | Nothing. Only the **default VPC** (`172.31.0.0/16`) remains. |
| **EC2 → Instances** | `terraweek` | No `terraweek-*-server`. A `terminated` server may show for a few minutes. |
| **EC2 → Security Groups** | `terraweek` | Nothing |
| **VPC → Internet gateways** | | Only the default VPC's gateway |

![Console VPCs clean](Screenshots/52-console-vpcs-clean.png)

![Console EC2 clean](Screenshots/53-console-ec2-clean.png)

### Is the AWS account completely clean?

All **21 capstone resources** are gone. But the account is **not empty**:
- The **default VPC** belongs to AWS and stays.
- Servers or buckets from other days are not part of this capstone. Check them separately.
- **EKS clusters are expensive.** If you built one on another day, make sure it is destroyed.

---

## Before you push to GitHub

State files can contain secrets. Keep them out of Git.

```bash
nano .gitignore
```

Paste:

```
.terraform/
*.tfstate
*.tfstate.*
terraform.tfstate.d/
```

- **Commit** `.terraform.lock.hcl` (it pins the provider version).
- The tfvars files contain no secrets, so they can be committed.

Check, then push:

```bash
git status         # no .tfstate files should be in the list
git add .
git commit -m "TerraWeek capstone: multi-environment infra with modules and workspaces"
git push
```

---

## Summary

| Task | What we did | Lesson |
|---|---|---|
| 1 | Created dev, staging, prod workspaces | Each workspace has its own state |
| 2 | Created folders and files | Clean structure = easy project |
| 3 | Wrote VPC, Security Group, EC2 modules | Write once, reuse everywhere |
| 4 | Wrote root files and 3 tfvars | Same code, different values per environment |
| 5 | Deployed 3 environments (21 resources) | State isolation keeps environments safe |
| 6 | Best practices guide | Pair workspace and tfvars, protect prod |
| 7 | Destroyed all, deleted workspaces, checked Console | Always clean up to avoid bills |

### Golden rules

1. **Workspace and tfvars must match.** Check with `terraform workspace show`.
2. **Plan before apply**, every time.
3. **Never commit state files** or secrets.
4. **Destroy test environments** when you finish.
5. **Modules do the work, the root module only passes values.**
