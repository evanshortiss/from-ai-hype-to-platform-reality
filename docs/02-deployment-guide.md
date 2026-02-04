# Deployment Guide

This guide provides step-by-step instructions for deploying the demo to your OpenShift cluster.

## Prerequisites

Before starting, ensure you have completed all steps in the [Getting Started Guide](01-getting-started.md).

> [!IMPORTANT]
> GitHub Organization Setup is required for the demo!

**Before deploying, ensure you have:**

1. ✅ Created or have access to a GitHub organization
2. ✅ Forked this repository to your organization
3. ✅ Created a GitHub App in your organization
4. ✅ Cloned your forked repository (not the original/upstream)

If you haven't completed these steps, go back to the [Getting Started Guide](01-getting-started.md#github-organization-setup).

**Why this matters:**
- RHDH will load catalog and templates from YOUR forked repository
- New services created from templates will go into YOUR organization
- GitHub App authentication requires YOUR organization

## Step 1: Configure the Inventory File

### Create Your Inventory

```bash
# From the repository root
cd deployment

# Copy the template to create your inventory
cp inventories/inventory.template inventories/inventory
```

### Edit the Inventory File

> [!NOTE]
> See [Inventory Configuration Reference](03-inventory-configuration.md) for complete variable documentation.

Open `inventories/inventory` in your preferred text editor. Update the following **required** variables:

#### GitHub Configuration

```ini
# GitHub organization where you forked this repository
# IMPORTANT: This must match the org where you created your GitHub App
# Example: acme-corp (if you forked to https://github.com/acme-corp/from-ai-hype-to-platform-reality)
github_organization=your-github-org

# GitHub App ID from the app settings page
github_app_id=123456

# GitHub OAuth Client ID from the app settings
github_client_id=Iv1.abc123def456

# GitHub OAuth Client Secret (generated in app settings)
github_client_secret=ghs_your_actual_client_secret_here

# Path to GitHub App private key PEM file
# Can be relative to deployment/ directory or absolute path
github_private_key_file=github-app-private-key.pem
```

#### LLM Configuration

```ini
# OpenAI API key or compatible endpoint key
default_llm_api_key=sk-your-api-key-here

# Model name (default works with OpenAI)
default_llm_model_name=gpt-4o-mini

# Base URL (default works with OpenAI)
default_llm_base_url=https://api.openai.com/v1
```

## Step 2: Authenticate to OpenShift

```bash
# Login to your cluster
oc login --server=https://api.YOUR-CLUSTER-DOMAIN.com:6443

# Verify authentication
oc whoami
oc cluster-info

# Verify cluster-admin access
oc auth can-i create namespace
# Should return: yes
```

## Step 3: Run the Ansible Playbook

### Deploy the Demo using Ansible

Run the playbook with `ACTION=create`:

```bash
ansible-playbook -i inventories/inventory playbooks/ocp4_workload_ai_summit2025.yml -e ACTION=create
```

### Access Argo CD

Get the Argo CD route:

```bash
oc get route -n openshift-gitops openshift-gitops-server -o jsonpath='{.spec.host}'
```

Get the Argo CD admin password:

```bash
oc extract secret/openshift-gitops-cluster -n openshift-gitops --to=-
```

Login to Argo CD:

- Username: `admin`
- Password: (from previous command)

### Monitor Application Sync

In the Argo CD UI, you should see applications for:
- Infrastructure components
- All "parasol" microservices
- Red Hat Developer Hub configuration

Wait for all applications to show "Healthy" and "Synced" status. If AMQ Streams or others fail sync, check the operator versions for a mismatch.

### Access UIs

Access the **AMQ Streams Console** to view Kafka topics:

```bash
# Get the Kafka UI route
oc get route -n kafka
```

Access Developer Hub in your browser:

> [!NOTE]
> You cannot login yet, you need to set the redirect URL for your GitHub App. That URL was printed in the Ansible output!


