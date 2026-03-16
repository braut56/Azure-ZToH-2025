1️  - Install and configure:

Terraform
Azure CLI

Azure account on Microsoft Azure

Login:
az login
Select subscription:

az account set --subscription "SUBSCRIPTION_ID"
2- Project Folder Structure
Create a Terraform project:
aks-terraform/
 provider.tf
 variables.tf
 main.tf
 outputs.tf
 terraform.tfvars

3️- provider.tf
Defines Terraform provider and version.
terraform {
  required_version = ">= 1.5"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~>3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

4️ -variables.tf
Define reusable variables.

variable "resource_group_name" {
  default = "aks-rg"
}

variable "location" {
  default = "East US"
}

variable "cluster_name" {
  default = "demo-aks"
}

variable "dns_prefix" {
  default = "demoaks"
}

variable "node_count" {
  default = 2
}

variable "vm_size" {
  default = "Standard_DS2_v2"
}

5️- main.tf
Creates Resource Group + AKS cluster.

resource "azurerm_resource_group" "aks_rg" {
  name     = var.resource_group_name
  location = var.location
}

resource "azurerm_kubernetes_cluster" "aks_cluster" {
  name                = var.cluster_name
  location            = azurerm_resource_group.aks_rg.location
  resource_group_name = azurerm_resource_group.aks_rg.name
  dns_prefix          = var.dns_prefix

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
  }

  tags = {
    environment = "dev"
  }
}
6- outputs.tf
Outputs useful values.
output "aks_cluster_name" {
  value = azurerm_kubernetes_cluster.aks_cluster.name
}

output "resource_group_name" {
  value = azurerm_resource_group.aks_rg.name
}
output "kube_config" {
  value     = azurerm_kubernetes_cluster.aks_cluster.kube_config_raw
  sensitive = true
}

7️ -terraform.tfvars
Override defaults.
resource_group_name = "aks-demo-rg"
location            = "East US"
cluster_name        = "demo-aks-cluster"
dns_prefix          = "demoaks"
node_count          = 2
vm_size             = "Standard_DS2_v2"

8️ - Initialize Terraform
terraform init
9️ - Validate configuration
terraform validate
10 - Plan deployment
terraform plan
11- Deploy AKS
terraform apply
Confirm with:
yes

1️2- Connect to AKS
After deployment:
az aks get-credentials \
  --resource-group aks-demo-rg \
  --name demo-aks-cluster

Test:
kubectl get nodes

You should see the nodes of your Kubernetes cluster running on Azure Kubernetes Service.
