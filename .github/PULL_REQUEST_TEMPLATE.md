## Type of Change

- [ ] Bug fix
- [ ] New feature or improvement
- [ ] Helm chart update
- [ ] Dockerfile update
- [ ] Documentation update
- [ ] Other (describe below)

## Description

<!-- What are you changing and why? -->

## Checklist

- [ ] No hardcoded credentials, tokens, or organization-specific values in `values.yaml`
- [ ] Dockerfile changes use Ubuntu LTS base image and clean up apt cache in the same RUN layer
- [ ] Helm chart changes are reflected in the README Configuration table
- [ ] Tested against a running AKS cluster with KEDA installed (if applicable)
- [ ] I have read [CONTRIBUTING.md](../CONTRIBUTING.md)
