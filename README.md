# Platform GitOps

This repository is a GitOps-based deployment platform for Kubernetes applications. It centralizes application definitions and Helm charts in version control, then uses Argo CD to reconcile the desired state in a target cluster automatically.

## Overview

The repository is organized around two core patterns:

- application Helm charts under the apps directory
- an Argo CD ApplicationSet that discovers Helm charts dynamically from the repository and creates deployable application resources

This approach makes it easy to add, version, and manage multiple workloads using a single GitOps workflow.

## Repository structure

```text
.
├── applicationsets/
│   └── applications.yaml
├── apps/
│   └── business-management/
│       ├── README.md
│       └── helm/
└── README.md
```

## Current application

| Application | Path | Type | Status |
| --- | --- | --- | --- |
| Business Management | [apps/business-management](apps/business-management) | Helm chart | Active |

## What this repo deploys

This repository is designed to host multiple application workloads, each packaged as an independent Helm chart and deployed through GitOps. The general application pattern includes:

- application workload deployment
- Kubernetes Service exposure
- optional Ingress for external access
- supporting data services such as MySQL or other backing dependencies
- secret synchronization through the External Secrets Operator
- integration with external secret stores such as AWS Secrets Manager
- environment-specific configuration managed through Helm values

Each app under the apps directory follows this structure, allowing the platform to scale as new services are added without changing the GitOps model.

## GitOps flow

The ApplicationSet in [applicationsets/applications.yaml](applicationsets/applications.yaml) watches the repository and automatically generates Argo CD Application resources for each Helm chart under the apps/*/helm path.

This enables a scalable structure where new applications can be added by creating a new folder under apps and placing a Helm chart inside it.

## Prerequisites

Before using this repository, ensure the following are available:

- Kubernetes cluster
- kubectl configured for the target cluster
- Helm 3 installed locally
- Argo CD installed and reachable in the cluster
- ingress controller (if ingress is enabled)
- External Secrets Operator installed
- AWS access configured for secret retrieval when using Secrets Manager
- container registry access for the application image


## Getting started

1. Clone the repository:

   ```bash
   git clone https://github.com/abdelrahmanelhabal/platform-gitops.git
   cd platform-gitops
   ```

2. Review the Helm chart and configuration for the application you want to deploy:

   ```bash
   ls apps
   cat apps/<app-name>/helm/values.yaml
   ```

3. Install or sync the ApplicationSet in Argo CD:

   ```bash
   kubectl apply -f applicationsets/applications.yaml
   ```

4. Confirm that Argo CD detects and syncs the applications defined under the apps directory.

## Application deployment

Each application in this repository is deployed as an independent Helm release using its own chart under the apps directory. The deployment model is consistent across services and can be scaled to add more workloads over time.

To deploy an application manually from the repository root:

```bash
helm upgrade --install <app-name> ./apps/<app-name>/helm \
  --namespace <app-namespace> \
  --create-namespace
```

For example, a chart under apps/business-management/helm can be installed as:

```bash
helm upgrade --install business-management ./apps/business-management/helm \
  --namespace business-management \
  --create-namespace
```

For full setup details, configuration options, and operational guidance for a specific service, refer to that service's README in its app folder.

## Security and configuration

This repository follows a GitOps and secret-management model aligned with Kubernetes best practices:

- application configuration is stored in Git
- environment values are managed through Helm values per application
- sensitive configuration can be sourced from external secret stores such as AWS Secrets Manager
- secret synchronization is handled through the External Secrets Operator
- cluster-level access and secret permissions are managed through the required Kubernetes and cloud identities
