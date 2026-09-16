# Business Management

This repository contains the deployment definition for the Business Management application using Helm. The chart packages the application, database, ingress exposure, and secret management into a single deployable unit for Kubernetes environments.

## Overview

The Business Management Helm chart deploys a production-ready application stack consisting of:

- the application container
- a Kubernetes `Service` for internal communication
- optional `Ingress` for external access
- a MySQL database deployed as a `StatefulSet`
- AWS Secrets Manager integration through the External Secrets operator

This is intended for a GitOps-style workflow, where application configuration and infrastructure dependencies are versioned together and deployed as a single release.

## Repository structure

```text
apps/
  business-management/
    README.md
    helm/
      Chart.yaml
      values.yaml
      templates/
        configmap.yaml
        deployment.yaml
        external-secret.yaml
        ingress.yaml
        mysql.yaml
        service.yaml
```

## Prerequisites

Before deploying this project, ensure the following prerequisites are satisfied:

- a reachable Kubernetes cluster
- Helm 3 installed locally
- `kubectl` configured for the target cluster
- an ingress controller installed if Ingress will be used
- the External Secrets Operator installed in the cluster
- valid AWS credentials with permission to read Secrets Manager entries
- the application Docker image published to a registry accessible by the cluster
- sufficient storage and compute resources for the MySQL workload

## Default chart configuration

The default values are defined in [apps/business-management/helm/values.yaml](apps/business-management/helm/values.yaml).

| Component | Default value |
| --- | --- |
| Application image | `abdelrahaman910/business-management-system:latest` |
| Service type | `ClusterIP` |
| Application port | `8080` |
| Database name | `businessproject` |
| MySQL image | `mysql:8.4` |
| Ingress host | `business-management.local` |
| AWS secret name | `business-management-mysql` |
| MySQL storage | `10Gi` |

## Security and secret management

This chart retrieves database credentials from AWS Secrets Manager using the External Secrets operator.

### 1. Create the AWS credential secret in the cluster

```bash
kubectl create secret generic aws-secret \
  --from-literal=access-key-id="YOUR_AWS_ACCESS_KEY_ID" \
  --from-literal=secret-access-key="YOUR_AWS_SECRET_ACCESS_KEY" \
  -n <namespace>
```

### 2. Create the corresponding AWS Secrets Manager entry

```bash
aws secretsmanager create-secret \
  --name business-management-mysql \
  --secret-string '{"DATABASE_USER":"appuser","DATABASE_PASSWORD":"your-db-password","DATABASE_NAME":"businessproject"}'
```

> The exact secret structure may differ depending on your AWS setup and environment conventions. The chart expects the database credential values required by the application and MySQL.

## Deployment instructions

From the repository root, deploy the release with:

```bash
helm upgrade --install business-management ./apps/business-management/helm \
  --namespace business-management \
  --create-namespace
```

### Customizing values

Create a custom values file and pass it during deployment:

```bash
helm upgrade --install business-management ./apps/business-management/helm \
  --namespace business-management \
  --create-namespace \
  -f my-values.yaml
```

Common customizations include:

- changing the image repository and tag
- enabling or disabling ingress
- modifying the application port or service type
- updating the database name and storage size
- adjusting the MySQL image version

## Validation and health checks

After deployment, verify that the resources are running correctly:

```bash
kubectl get pods -n business-management
kubectl get svc -n business-management
kubectl get ingress -n business-management
kubectl get pvc -n business-management
```

Render the manifests without applying them to confirm the chart output:

```bash
helm template business-management ./apps/business-management/helm -n business-management
```

## Accessing the application

If ingress is enabled, access the application at:

```text
http://business-management.local
```

For local development, add the following entry to your `/etc/hosts` file:

```bash
127.0.0.1 business-management.local
```

If you do not want to use ingress, expose the service locally with port forwarding:

```bash
kubectl port-forward svc/business-management -n business-management 8080:80
```

Then open:

```text
http://localhost:8080
```

## Useful operational commands

```bash
# list all resources in the namespace
kubectl get all -n business-management

# check application logs
kubectl logs deploy/business-management -n business-management

# check MySQL logs
kubectl logs statefulset/business-management-mysql -n business-management

# review the release history
helm history business-management -n business-management

# rollback to a previous revision
helm rollback business-management <REVISION> -n business-management

# uninstall the release
helm uninstall business-management -n business-management
```

## Upgrade and rollback considerations

When upgrading the chart:

- review any changes in the values file before applying
- validate the rendered manifests with `helm template`
- confirm the new image is available and compatible with the database schema
- verify that persistent volumes and secrets remain valid across upgrades

For production environments, use a controlled release process and test changes in a lower environment before promoting them.

## Notes and operational considerations

- The application connects to MySQL using the hostname generated by the chart.
- Application and database credentials are managed through Kubernetes Secrets populated by External Secrets.
- MySQL is configured with persistent storage and is deployed as a StatefulSet for stable identity and storage behavior.
- The chart assumes a valid Kubernetes cluster, secure registry access, and AWS permissions in the target environment.
- Ingress configuration and DNS settings must be prepared before exposing the application externally.

## Troubleshooting

If deployment fails, verify the following points:

1. the External Secrets CRDs are installed and healthy
2. AWS credentials are valid and the secret exists in AWS Secrets Manager
3. the application image is available in the container registry
4. the cluster has sufficient resources for application and database pods
5. the ingress controller is installed if you are using the default ingress host
6. the database hostname and credentials match the application configuration

To inspect the rendered manifests before deployment:

```bash
helm template business-management ./apps/business-management/helm -n business-management
```

This helps identify invalid configuration, template errors, or secret reference issues before the release is applied.

## Production readiness checklist

Before promoting to a production environment, confirm the following:

- Secrets are managed securely and not stored in plain manifests
- Ingress TLS and DNS are configured correctly
- the database is backed by persistent storage with a retention strategy
- monitoring and log collection are enabled for both the app and MySQL
- resource requests and limits are defined appropriately
- an upgrade and rollback plan is established for the release


