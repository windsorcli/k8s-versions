# Kubernetes CSP Versions

This repository tracks the latest supported Kubernetes versions for various Cloud Service Providers (CSPs). It's designed to work with Renovate for automated version updates.

## Supported CSPs

- Amazon EKS (Elastic Kubernetes Service)
- Azure AKS (Azure Kubernetes Service)

## How it Works

A GitHub Action runs daily to:
1. Scrape supported Kubernetes versions from each CSP's API
2. Create Git tags with the format `{csp}-{version}` (e.g., `eks-1.28`, `aks-1.27`)
3. Force push these tags to the repository

## Usage with Renovate

Configure Renovate to track these tags in your repository's `renovate.json`:

```json
{
  "packageRules": [
    {
      "matchPackageNames": ["kubernetes"],
      "matchManagers": ["helm"],
      "matchUpdateTypes": ["version"],
      "allowedVersions": "regex:^(eks|aks)-.*$"
    }
  ]
}
```

## Manual Trigger

The GitHub Action can be manually triggered from the Actions tab in the repository.
