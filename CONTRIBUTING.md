# Contributing to AKS Azure DevOps Agents

Thank you for your interest in contributing!

## How to Contribute

### Reporting Issues

Use GitHub Issues to:
- Report bugs or misconfigured defaults
- Suggest improvements to the Helm chart or Dockerfile
- Request support for additional Kubernetes features

### Proposing Changes

1. Fork this repository
2. Create a feature branch: `git checkout -b feat/description` or `fix/description`
3. Make your changes
4. Submit a Pull Request using the provided template

### Guidelines

- **Dockerfile**: Use the current Ubuntu LTS base image; combine `RUN` steps to minimize layers; clean up `apt` cache in the same layer
- **Helm chart**: Follow Helm best practices — use `values.yaml` for all configurable parameters; do not hardcode organization-specific values in templates
- **Security**: Never commit real credentials, tokens, or ACR URLs in `values.yaml` — use placeholder strings
- **Backwards compatibility**: If changing `values.yaml` structure, update the README Configuration table accordingly
- **KEDA**: Reference the [KEDA Azure Pipelines scaler documentation](https://keda.sh/docs/latest/scalers/azure-pipelines/) for scaler-specific options

## Code of Conduct

Please read and follow our [Code of Conduct](CODE_OF_CONDUCT.md).
