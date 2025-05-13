---
title: Getting started with Terraform
date: 2025-05-13 03:38:57
tags: [terraform, automation, opentofu, IaaC, proxmox]
---

At this point I have never had any real hands-on experience with [Terraform](https://developer.hashicorp.com/terraform) - I have been aware of it and what it does, but never actually used it. Now, however I feel like I want to automate my homelab setup one step further, and I believe starting from the infrastructure is as good place to start as any. Plus, I like the idea of having my homelab setup in declarative form for the future reference.

As I already use [Semaphore UI](https://semaphoreui.com/) for running some of my [Ansible](https://www.redhat.com/en/ansible-collaborative) playbooks, I will be using it to run Terraform/[OpenTofu](https://opentofu.org/) as well.

## Preparing for Terraform

To keep my Terraform files separate from everything else, I created a new repository in my self-hosted [Gitea](https://about.gitea.com/) instance, and added the details into the Semaphore UI.

## Starting with Terraform

#### providers.tf

> To get the variables from Semaphore UI Variable Groups, the vars should include `TF_VAR_` prefix. For instance in the `provider.tf` file, a variable is referenced as `pm_api_url` and in Semaphore UI, the same variable is `TF_VAR_pm_api_url`.
{: .prompt-warning }

Create a new file `provider.tf`. This file will include the [provider](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs) details and the Proxmox instance details.

```terraform
terraform {
  required_providers {
    proxmox = {
      source = "Telmate/proxmox"
      version = ">= 2.9.0"
    }
  }
}

variable "pm_api_url" {
  type = string
}

variable "pm_api_id" {
  type = string
}

variable "pm_api_token" {
  type = string
  sensitive = true
}

provider "proxmox" {
  pm_api_url = var.pm_api_url
  pm_api_token_id = var.pm_api_id
  pm_api_token_secret = var.pm_api_token

  # Set this true only when Proxmox host is using self-signed certificates.
  pm_tls_insecure = true
}
```

### Module for LXC containers

I wanted as little repetition as possible, so I opted for using a module to define the logic for LXC container setup. This way I can have simpler files in the root folder defining the containers themselves.

Create a folder called `modules` and inside it a folder called `lxc`. Add four files (`main.tf`, `outputs.tf`, `variables.tf`, and `versions.tf`) into the module folder.

> Details about available variables for the module can be found in the [provider registry page](https://registry.terraform.io/providers/Telmate/proxmox/latest/docs/resources/lxc).
{: .prompt-tip }

#### main.tf

```terraform
resource "proxmox_lxc" "container" {
  features {
      nesting = var.nesting
  }

  unprivileged = var.unprivileged
  onboot = var.onboot
  start = var.start

  rootfs {
    storage = var.disk_storage
    size    = var.disk_size
  }

  network {
    name = var.network_name
    bridge = var.network_bridge
    ip = var.network_ip
    gw = var.network_gw
    tag = var.network_tag
  }

  nameserver = var.network_ns
  
  ssh_public_keys = var.ssh_keys
  password = var.lxc_password
  target_node = var.target_node
  hostname = var.hostname
  ostemplate = var.ostemplate
  vmid = var.vmid
  cores = var.cores
  memory = var.memory
  swap = var.swap
}
```

#### outputs.tf

```terraform
output "lxc_ip" {
  value = proxmox_lxc.container.network[0].ip
}
```

#### variables.tf

> In this file, default values can be set. The variables without default values would need to be defined in the file for the actual container definition. E.g. [100-my-important-ct.tf](#defining-lxc-containers).
{: .prompt-info }

```terraform
variable "nesting" {
  default = true
}

variable "unprivileged" {
  default = true
}
variable "onboot" {
  default = false
}
variable "start" {
  default = true
}

variable "disk_storage" {
  default = "local"
}
variable "disk_size" {
  default = "8G"
}

variable "network_name" {
  default = "eth0"
}
variable "network_bridge" {
  default = "vmbr0"
}
variable "network_ip" {
  default = "dhcp"
}
variable "network_gw" {
  default = ""
}
variable "network_tag" {
  default = null
}

variable "network_ns" {
  default = ""
}

variable "ssh_keys" {
  default = ""
}
variable "lxc_password" {}
variable "target_node" {}
variable "hostname" {}
variable "ostemplate" {
  default = "local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst"
}
variable "vmid" {
  default = 0
}
variable "cores" {
  default = 1
}
variable "memory" {
  default = 512
}
variable "swap" {
  default = 512
}
```

#### versions.tf

> Initially I tried without `versions.tf` file, however when running the task in Semaphore UI, OpenTofu was trying to pull a wrong provider and I kept getting errors. After some trial and error as well as research and consultation with an LLM, I found a solution that worked for me, even if it is a bit of code repetition.
{: .prompt-info }

```terraform
terraform {
  required_providers {
    proxmox = {
      source = "Telmate/proxmox"
      version = ">= 2.9.0"
    }
  }
}
```

### Defining LXC containers

To keep things better organised for myself and to avoid having a massive file to include all the lxc containers I want Terraform/OpenTofu to create, I will have a separate file in the root folder with the name of the container. E.g. `100-my-important-ct.tf`.

> Variables that don't have defaults defined in `./modules/lxc/variables.tf` will have to be defined regardless, others with defaults only when override is needed. E.g. more memory or cores etc.
{: .prompt-info }

```terraform
module "lxc" {
  source = "./modules/lxc"
  target_node = "proxmox"
  hostname = "my-important-ct"
  vmid = 100
  cores = 2
  memory = 2048
  disk_size = "16G"
  lxc_password = "very-secure-root-password"
}
```

**_Voilà!_**