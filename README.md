# Azure DevOps Agents on AKS with KEDA Autoscaling
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This repository provides a Helm chart for deploying self-hosted Azure DevOps agents on Azure Kubernetes Service (AKS) with KEDA-based autoscaling. The solution ensures efficient resource utilization by scaling the agents dynamically based on pending jobs in the Azure DevOps pipeline.

## Features

- **Docker-based agent deployment** — Ubuntu 24.04 LTS base image with Azure Pipelines agent
- **Helm chart for easy deployment** — deploy with configurable parameters via `values.yaml`
- **KEDA Autoscaling** — dynamically scales agents based on Azure DevOps pipeline queue length
- **Secure authentication** — uses Kubernetes Secrets for storing Azure DevOps PAT
- **Customizable configurations** — agent pool, image version, and scaling limits are all configurable

## Repository Structure

```
aks-azure-devops-agents/
├── docker/
│   ├── Dockerfile              # Agent container image (Ubuntu 24.04 LTS)
│   └── start.sh                # Agent startup and registration script
├── helm-chart/
│   └── aks-azure-devops-agents/
│       ├── templates/
│       │   ├── deployment.yaml      # Kubernetes Deployment
│       │   ├── scaledobject.yaml    # KEDA ScaledObject
│       │   ├── secret.yaml          # Azure DevOps PAT secret
│       │   └── trigger-auth.yaml    # KEDA TriggerAuthentication
│       ├── Chart.yaml
│       └── values.yaml
└── pipelines/
    └── azure-pipelines.yaml    # CI/CD pipeline for building and pushing the image
```

## Prerequisites

- **Azure Kubernetes Service (AKS)** cluster
- **Helm v3** installed
- **KEDA installed on AKS** ([KEDA installation guide](https://keda.sh/docs/latest/deploy/))
- **Azure DevOps account with an agent pool**
- **Personal Access Token (PAT)** with Agent Pools (Read & Manage) permission

## Building and Pushing the Docker Image

1. Clone the repository:
   ```bash
   git clone https://github.com/Think-Cube/aks-azure-devops-agents.git
   cd aks-azure-devops-agents/docker
   ```
2. Build the Docker image:
   ```bash
   docker build -t <your-acr>.azurecr.io/aks-azure-devops-agents:<tag> .
   ```
3. Push the image to Azure Container Registry (ACR):
   ```bash
   az acr login --name <your-acr>
   docker push <your-acr>.azurecr.io/aks-azure-devops-agents:<tag>
   ```

## Deploying with Helm

1. Copy and edit the values file with your settings:
   ```bash
   cp helm-chart/aks-azure-devops-agents/values.yaml my-values.yaml
   # edit my-values.yaml: set image.repository, azp.url, azp.pool, azp.token, scaledObject.poolID
   ```
2. Create Kubernetes namespace:
   ```bash
   kubectl create namespace devops-agents
   ```
3. Deploy using Helm:
   ```bash
   helm install azdevops-agents helm-chart/aks-azure-devops-agents \
     --namespace devops-agents \
     --values my-values.yaml
   ```

## Scaling Behavior

KEDA monitors the number of pending jobs in the Azure DevOps agent pool and automatically scales the number of running agents between `minReplicaCount` and `maxReplicaCount`. Scaling is managed via the `ScaledObject` resource in `templates/scaledobject.yaml`.

## Upgrading

```bash
helm upgrade azdevops-agents helm-chart/aks-azure-devops-agents \
  --namespace devops-agents \
  --values my-values.yaml
```

## Uninstalling

```bash
helm uninstall azdevops-agents --namespace devops-agents
```

## Configuration

| Parameter | Description | Default |
|---|---|---|
| `replicaCount` | Initial number of agent replicas | `1` |
| `image.repository` | Docker image repository | `<your-acr>.azurecr.io/aks-azure-devops-agents` |
| `image.tag` | Image tag | `latest` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `azp.url` | Azure DevOps organization URL | `https://dev.azure.com/<your-org>` |
| `azp.pool` | Azure DevOps agent pool name | `<your-agent-pool>` |
| `azp.token` | Base64-encoded Azure DevOps PAT | `""` |
| `dockerVolumePath` | Docker socket path on host | `/var/run/docker.sock` |
| `scaledObject.minReplicaCount` | Minimum running agents | `1` |
| `scaledObject.maxReplicaCount` | Maximum running agents | `5` |
| `scaledObject.poolID` | Azure DevOps agent pool numeric ID | `<your-pool-id>` |
| `triggerAuthentication.name` | KEDA TriggerAuthentication resource name | `pipeline-trigger-auth` |

> **Finding the pool ID:** Go to Organization Settings → Agent pools → select your pool → the `poolId` query parameter in the URL is the numeric ID.

## Troubleshooting

### Check agent logs
```bash
kubectl logs -f deployment/azdevops-deployment -n devops-agents
```

### Debug KEDA scaling
```bash
kubectl get scaledobjects -n devops-agents
kubectl describe scaledobject azure-pipelines-scaledobject -n devops-agents
```

### Agent not registering
- Verify `azp.url` matches your Azure DevOps organization URL exactly
- Verify the PAT is correctly Base64-encoded and has Agent Pools (Read & Manage) permission
- Check that the pool name in `azp.pool` matches the pool created in Azure DevOps

## License

MIT © [Think-Cube](https://github.com/Think-Cube)

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.
