# AKS Cluster creation using Terraform

1. Create application registration (say terraform) Select EntraId resource and create AppRegistration with a Client Secret
2. Goto Subscription and give contributor role to the terraform app registration.

```
# terraform {
#   required_providers {
#     azurerm = {
#       source = "hashicorp/azurerm"
#       version = "3.97.1"
#     }
#   }
# }

# provider "azurerm" {

# subscription_id = "50632fd4-8723-443d-b2c4-58b2921778681"
# client_id = "ed6f9186-bbfa-4402-82af-8c0aec81dc031"  
# client_secret = "iji8Q~-QQcJ0YtcD.WEDpYSFbSgnZNpppyMKpb4u1"
# tenant_id = "bc12bee3-2ebd-4a4c-bf6f-1b4bf4d429c1"
# features {}

# }
# resource "azurerm_resource_group" "azurerg" {
#   name = "azurerg"
#   location = "East US"
# }

# Define Azure provider
provider "azurerm" {
  features {}
}

# Variables
variable "resource_group_name" {
  default = "myResourceGroup"
}

variable "location" {
  default = "East US"
}

variable "aks_cluster_name" {
  default = "myAKSCluster"
}

variable "aks_node_count" {
  default = 3
}

variable "aks_node_vm_size" {
  default = "Standard_DS2_v2"
}

# Resource Group
resource "azurerm_resource_group" "rg" {
  name     = var.resource_group_name
  location = var.location
}

# Virtual Network
resource "azurerm_virtual_network" "vnet" {
  name                = "${var.aks_cluster_name}-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = var.location
  resource_group_name = azurerm_resource_group.rg.name
}

# Subnet
resource "azurerm_subnet" "subnet" {
  name                 = "${var.aks_cluster_name}-subnet"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

# AKS Cluster
resource "azurerm_kubernetes_cluster" "aks" {
  name                = var.aks_cluster_name
  location            = var.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix          = "${var.aks_cluster_name}-dns"

  default_node_pool {
    name       = "default"
    node_count = var.aks_node_count
    vm_size    = var.aks_node_vm_size
  }


  service_principal {
    client_id     = "ed6f9186-bbfa-4402-82af-8c0aec81dc031"
    client_secret = "iji8Q~-QQcJ0YtcD.WEDpYSFbSgnZNpppyMKpb4u1"
  }

  tags = {
    Environment = "Production"
  }
}

# output "kube_config" {
#   value = azurerm_kubernetes_cluster.aks.kube_config_raw
# }

```
