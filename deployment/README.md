# Deployment Automation

This directory contains Ansible automation for deploying the Red Hat One demonstration platform to OpenShift.

## Quick Start

```bash
# From this directory
cd deployment

# Configure your inventory
cp inventories/inventory.template inventories/inventory
vi inventories/inventory  # Edit with your configuration

# Authenticate to OpenShift
oc login --server=https://api.your-cluster.com:6443

# Deploy the platform
ansible-playbook -i inventories/inventory playbooks/ocp4_workload_ai_summit2025.yml -e ACTION=create
```

## Contents

### Playbooks

- **playbooks/ocp4_workload_ai_summit2025.yml** - Main deployment playbook

### Roles

- **roles/ocp4_workload_ai_summit2025/** - Complete deployment role including:
  - Operator installation (GitOps, AMQ Streams, RHDH)
  - Namespace creation
  - Kafka cluster and topic configuration
  - Milvus vector database deployment
  - Red Hat Developer Hub configuration
  - ArgoCD application deployment
  - Secret management

### Inventories

- **inventories/inventory.template** - Template with all configuration variables
- **inventories/inventory** - Your local configuration (create from template)

### Action Plugins

- **action_plugins/** - Custom Ansible action plugins

### Lookup Plugins

- **lookup_plugins/** - Custom Ansible lookup plugins

## Configuration

See the [Inventory Configuration Reference](../docs/03-inventory-configuration.md) for complete documentation of all variables.

### Required Variables

At minimum, configure these in `inventories/inventory`:

```ini
# GitHub configuration
github_organization=evanshortiss
github_app_id=YOUR_APP_ID
github_client_id=YOUR_CLIENT_ID
github_client_secret=YOUR_CLIENT_SECRET
github_private_key_file=github-app-private-key.pem

# LLM configuration
default_llm_api_key=YOUR_API_KEY
default_llm_model_name=gpt-4o-mini
default_llm_base_url=https://api.openai.com/v1

# OpenShift certificate
openshift_ingress_secret_certs_secret=letsencrypt-certs
```

## Deployment Actions

### Create (Deploy)

Deploy the entire platform:

```bash
ansible-playbook -i inventories/inventory playbooks/ocp4_workload_ai_summit2025.yml -e ACTION=create
```

### Remove (Cleanup)

Remove all deployed resources:

```bash
ansible-playbook -i inventories/inventory playbooks/ocp4_workload_ai_summit2025.yml -e ACTION=remove
```

This will delete:
- All namespaces (parasol, parasol-ai, kafka, ai-rhdh, milvus)
- All ArgoCD applications
- All deployed services
- Kafka clusters and topics

**Note:** Operators (GitOps, AMQ Streams, RHDH) are NOT removed.

## What Gets Deployed

### Infrastructure

1. **OpenShift GitOps (ArgoCD)** - GitOps automation
2. **AMQ Streams (Kafka)** - Event streaming
3. **Red Hat Developer Hub** - Service catalog and templates
4. **Milvus** - Vector database
5. **Model Registry** - Model versioning

### Namespaces

- `parasol` - Demo services
- `parasol-ai` - AI demo services
- `kafka` - Kafka cluster and topics
- `ai-rhdh` - Red Hat Developer Hub
- `milvus` - Milvus vector database
- `openshift-gitops` - ArgoCD applications

### Kafka Topics

10 topics for service communication:
- moderation-input
- structured-output-input
- router-input
- finance-input
- support-input
- website-input
- finance-output
- support-output
- website-output
- routing-failure

### Microservices (via ArgoCD)

11 AI-powered services deployed via GitOps:
1. Entry Service
2. Moderation Service
3. Structured Output Service
4. Router Service
5. Finance Agent
6. Support Agent
7. Website Agent
8. Document Embedding Service
9. OpenAI Embedding Service
10. LLM Model Server
11. Analysis Agent

## Verification

After deployment, verify the platform:

```bash
# Check ArgoCD applications
oc get applications -n openshift-gitops

# Check Kafka topics (or use AMQ Streams Console)
oc get kafkatopic -n kafka

# Check service pods
oc get pods -n parasol
oc get pods -n parasol-ai

# Get RHDH URL
oc get route -n ai-rhdh backstage-developer-hub -o jsonpath='{.spec.host}'
```

All ArgoCD applications should show "Synced" and "Healthy" status.

## Troubleshooting

### Syntax Check

Verify playbook syntax before running:

```bash
ansible-playbook -i inventories/inventory playbooks/ocp4_workload_ai_summit2025.yml --syntax-check
```

### Verbose Output

Run with verbose output for debugging:

```bash
ansible-playbook -i inventories/inventory playbooks/ocp4_workload_ai_summit2025.yml -e ACTION=create -vvv
```

### Common Issues

**GitHub App Configuration**
- Verify all GitHub variables are correct
- Ensure private key file exists and is readable
- Check that the GitHub App is installed to your organization

**LLM API Errors**
- Verify API key is valid and active
- Check that model name is correct
- Ensure base URL includes `/v1` for OpenAI

**Operator Installation Failures**
- Check cluster permissions (cluster-admin required)
- Verify OperatorHub is accessible
- Check for existing operators (set install flags to false if present)

**Pod Failures**
- Check logs: `oc logs <pod-name> -n <namespace>`
- Describe pod: `oc describe pod <pod-name> -n <namespace>`
- Verify secrets are created correctly

## Documentation

For detailed documentation, see:

- [Getting Started Guide](../docs/01-getting-started.md) - Prerequisites and setup
- [Deployment Guide](../docs/02-deployment-guide.md) - Step-by-step walkthrough
- [Inventory Configuration](../docs/03-inventory-configuration.md) - Variable reference
- [Architecture Overview](../docs/04-architecture-overview.md) - System design

## Support

For issues and questions, open an issue in the GitHub repository.
