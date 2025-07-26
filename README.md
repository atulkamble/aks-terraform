**AKS (Azure Kubernetes Service) Deployment using Terraform**, including:

* **Terraform directory structure**
* **Code to provision** AKS with a node pool
* **How to use** `kubectl` to access the AKS cluster
* **Prerequisites** & Steps

---

## ✅ Prerequisites

* Azure CLI logged in: `az login`
* Terraform installed: `terraform -v`
* Azure subscription ID: `az account show --query id -o tsv`
* Create an Azure AD service principal for AKS authentication:

  ```bash
  az ad sp create-for-rbac --name terraform-aks-sp --skip-assignment
  ```

---

## 📁 Directory Structure

```
aks-terraform/
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
```

---

## 📄 main.tf

```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "aks_rg" {
  name     = var.resource_group_name
  location = var.location
}

resource "azurerm_kubernetes_cluster" "aks_cluster" {
  name                = var.cluster_name
  location            = azurerm_resource_group.aks_rg.location
  resource_group_name = azurerm_resource_group.aks_rg.name
  dns_prefix          = "aksdns-${random_id.dns.hex}"

  default_node_pool {
    name       = "default"
    node_count = var.node_count
    vm_size    = var.vm_size
  }

  identity {
    type = "SystemAssigned"
  }

  network_profile {
    network_plugin = "azure"
    load_balancer_sku = "standard"
  }

  tags = {
    Environment = "Dev"
  }
}

resource "random_id" "dns" {
  byte_length = 4
}
```

---

## 📄 variables.tf

```hcl
variable "resource_group_name" {
  description = "Azure Resource Group name"
  type        = string
  default     = "aks-terraform-rg"
}

variable "location" {
  description = "Azure Region"
  type        = string
  default     = "East US"
}

variable "cluster_name" {
  description = "AKS Cluster Name"
  type        = string
  default     = "aks-cluster"
}

variable "node_count" {
  description = "Number of nodes"
  type        = number
  default     = 2
}

variable "vm_size" {
  description = "VM Size for nodes"
  type        = string
  default     = "Standard_DS2_v2"
}
```

---

## 📄 outputs.tf

```hcl
output "kube_config" {
  value     = azurerm_kubernetes_cluster.aks_cluster.kube_config_raw
  sensitive = true
}

output "kube_config_path" {
  value = "${path.module}/kubeconfig"
}
```

---

## 📄 terraform.tfvars (optional)

```hcl
resource_group_name = "my-aks-rg"
location            = "East US"
cluster_name        = "my-aks-cluster"
node_count          = 2
vm_size             = "Standard_DS2_v2"
```

---

## 🚀 Terraform Deployment Steps

```bash
# Go to directory
cd aks-terraform

# Initialize Terraform
terraform init

# Validate configuration
terraform validate

# Preview the plan
terraform plan

# Apply the configuration
terraform apply -auto-approve
```

---

## 📦 Access AKS Cluster with `kubectl`

```bash
# Save kubeconfig file
terraform output -raw kube_config > kubeconfig

# Set KUBECONFIG to the file
export KUBECONFIG=./kubeconfig

# Test cluster access
kubectl get nodes
```

---
