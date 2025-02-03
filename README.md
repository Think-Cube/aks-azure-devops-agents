# Azure DevOps Agents on AKS with KEDA Autoscaling

## Overview
This repository provides a Helm chart for deploying Azure DevOps agents on Azure Kubernetes Service (AKS) with KEDA-based autoscaling. The solution ensures efficient resource utilization by scaling the agents dynamically based on pending jobs in the Azure DevOps pipeline.

## Features
- **Docker-based agent deployment**: Uses a custom Ubuntu-based Docker image.
- **Helm chart for easy deployment**: Deploy the agent using Helm with configurable parameters.
- **KEDA Autoscaling**: Dynamically scales agents based on Azure DevOps pipeline queue length.
- **Secure authentication**: Uses Kubernetes Secrets for storing Azure DevOps Personal Access Token (PAT).
- **Customizable configurations**: Helm values allow flexibility in agent pool selection, image versions, and scaling limits.

## Repository Structure
```
aks-azure-devops-agents/
│── docker/
│   ├── Dockerfile        # Dockerfile for the agent container
│   ├── start.sh          # Startup script for the agent
│
│── helm-chart/
│   ├── aks-azure-devops-agents/
│   │   ├── templates/
│   │   │   ├── deployment.yaml       # Deployment configuration
│   │   │   ├── scaledobject.yaml     # KEDA scaling definition
│   │   │   ├── secret.yaml           # Azure DevOps authentication secret
│   │   │   ├── trigger-auth.yaml     # KEDA trigger authentication
│   │   ├── Chart.yaml     # Helm chart metadata
│   │   ├── values.yaml    # Default configuration values
```

## Prerequisites
- **Azure Kubernetes Service (AKS)** cluster
- **Helm v3** installed
- **KEDA installed on AKS** ([KEDA installation guide](https://keda.sh/docs/latest/deploy/))
- **Azure DevOps account with an agent pool**
- **Personal Access Token (PAT) with agent management permissions**

## Building and Pushing the Docker Image
1. Clone the repository:
   ```bash
   git clone https://github.com/Think-Cube/aks-azure-devops-agents.git
   cd aks-azure-devops-agents/docker
   ```
2. Build the Docker image:
   ```bash
   docker build -t <your-acr>.azurecr.io/keda-devops-agents:<tag> .
   ```
3. Push the image to Azure Container Registry (ACR):
   ```bash
   az acr login --name <your-acr>
   docker push <your-acr>.azurecr.io/keda-devops-agents:<tag>
   ```

## Deploying with Helm
1. **Update Helm values (optional):** Modify `helm-chart/aks-azure-devops-agents/values.yaml` with your settings.
2. **Create Kubernetes namespace (if needed):**
   ```bash
   kubectl create namespace devops-agents
   ```
3. **Deploy using Helm:**
   ```bash
   helm install azdevops-agents helm-chart/aks-azure-devops-agents \
     --namespace devops-agents
   ```

## Scaling Behavior
KEDA will monitor the number of pending jobs in Azure DevOps and automatically scale the number of running agents between `minReplicaCount` and `maxReplicaCount`. Scaling is managed via the `ScaledObject` resource in `scaledobject.yaml`.

## Uninstalling the Deployment
To remove the agents and associated resources:
```bash
helm uninstall azdevops-agents --namespace devops-agents
```

## Configuration
| Parameter                 | Description                                      | Default |
|---------------------------|--------------------------------------------------|---------|
| `replicaCount`            | Initial number of agent replicas                | `1`     |
| `image.repository`        | Docker image repository                         | `your-acr.azurecr.io/keda-devops-agents` |
| `image.tag`               | Image tag                                       | `latest`|
| `image.pullPolicy`        | Image pull policy                              | `IfNotPresent` |
| `azp.url`                 | Azure DevOps organization URL                   | `"https://dev.azure.com/Test-Lab"` |
| `azp.pool`                | Azure DevOps agent pool name                    | `"Keda-Devops-Agent"` |
| `azp.token`               | Base64-encoded Azure DevOps PAT                 | (set via Secret) |
| `dockerVolumePath`        | Path to Docker socket for agent containers      | `"/var/run/docker.sock"` |
| `scaledObject.minReplicaCount` | Minimum number of running agents       | `1` |
| `scaledObject.maxReplicaCount` | Maximum number of running agents       | `5` |
| `scaledObject.poolID`     | Azure DevOps agent pool ID                      | `12` |
| `triggerAuthentication.name` | KEDA TriggerAuthentication name               | `pipeline-trigger-auth` |

## Troubleshooting
### Check Logs
```bash
kubectl logs -f deployment/azdevops-deployment -n devops-agents
```
### Debug Scaling Issues
```bash
kubectl get scaledobjects -n devops-agents
kubectl describe scaledobject azure-pipelines-scaledobject -n devops-agents
```

## License
This project is licensed under the MIT License.

