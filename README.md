# Kubernetes CSP Versions

This repository (`windsorcli/k8s-versions`) tracks the latest supported Kubernetes versions for various Cloud Service Providers (CSPs). It's designed to work seamlessly with Renovate for automated version updates in your infrastructure code.

## Supported CSPs

- Amazon EKS (Elastic Kubernetes Service)
- Azure AKS (Azure Kubernetes Service)

## How it Works

A GitHub Action runs daily to:
1. Scrape supported Kubernetes versions from each CSP's API
2. Create Git tags with the format `{csp}-kubernetes-{major}.{minor}` (e.g., `eks-kubernetes-1.28`, `aks-kubernetes-1.27`)
3. Push these tags to the repository

**Note on Regional Availability**: This repository tracks versions from a single reference region for each CSP. While CSPs roll out new versions across regions over time, we maintain consistency by checking the same region for each provider. This provides a reliable leading indicator of version availability.


## Usage with Renovate

Configure Renovate to track these tags in your Terraform or other IaC configurations. This allows you to automatically update your Kubernetes version references according to CSP-supported versions.

### Renovate Configuration Example

Below is an example configuration for your `renovate.json` file:

```json
{
  "packageRules": [
    {
      "matchPackageNames": ["windsorcli/k8s-versions"],
      "matchDepNames": ["aks-kubernetes"],
      "extractVersion": "^aks-kubernetes-(?<version>\\d+\\.\\d+)$",
      "versioning": "semver"
    },
    {
      "matchPackageNames": ["windsorcli/k8s-versions"],
      "matchDepNames": ["eks-kubernetes"],
      "extractVersion": "^eks-kubernetes-(?<version>\\d+\\.\\d+)$",
      "versioning": "semver"
    }
  ],
  "regexManagers": [
    {
      "fileMatch": ["(^|/).*\\.tf$"],
      "matchStrings": [
        "\\s*(?:\\w+_)?version\\s*=\\s*\"(?<currentValue>[^\"]+)\"\\s*#\\s*renovate:\\s*datasource=(?<datasource>[^\\s]+)\\s+depName=(?<depName>[^\\s]+)(?:\\s+package=(?<packageName>[^\\s]+))?\\s*"
      ],
      "datasourceTemplate": "{{datasource}}",
      "versioningTemplate": "semver"
    }
  ]
}
```

### In Your Terraform Files

Here are examples of how to reference the versions in your Terraform files:

#### AWS EKS Example

```hcl
resource "aws_eks_cluster" "eks_cluster" {
  name     = "eks-cluster"
  role_arn = aws_iam_role.eks_cluster_role.arn
  
  # Track specific CSP-supported version
  version = "1.28" # renovate: datasource=github-tags depName=eks-kubernetes package=windsorcli/k8s-versions
  
  vpc_config {
    subnet_ids = [aws_subnet.subnet1.id, aws_subnet.subnet2.id]
  }
}
```

#### Azure AKS Example

```hcl
resource "azurerm_kubernetes_cluster" "aks_cluster" {
  name                = "aks-cluster"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix          = "aks-cluster"
  
  # Track specific CSP-supported version
  kubernetes_version = "1.27" # renovate: datasource=github-tags depName=aks-kubernetes package=windsorcli/k8s-versions
  
  default_node_pool {
    name       = "default"
    node_count = 2
    vm_size    = "Standard_D2_v2"
  }

  identity {
    type = "SystemAssigned"
  }
}
```

## Available Tags

You can see all available version tags in the [Tags section](https://github.com/windsorcli/k8s-versions/tags) of this repository.

## Version Tag Format

The repository uses the following tag format for tracking Kubernetes versions:

- Standard versions: `{csp}-kubernetes-{version}`
- Preview versions: `{csp}-kubernetes-{version}-preview`

Where:
- `{csp}` is the cloud service provider (e.g., `eks`, `aks`)
- `{version}` is the Kubernetes version in major.minor format

Example tags:
- `eks-kubernetes-1.27`
- `eks-kubernetes-1.28-preview`
- `aks-kubernetes-1.27`
- `aks-kubernetes-1.28-preview`

These tags are automatically created and updated by the GitHub Action workflow.