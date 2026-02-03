# Getting Started Guide

This guide walks you through preparing your environment to deploy the Red Hat One demonstration platform. While this seems like a lot of steps upfront, the good news is you can reuse the GitHub Org again, and again, and again - it's a one time task.

Once all prerequisites are in place, proceed to the [Deployment Guide](02-deployment-guide.md) to configure and deploy the platform.

## Prerequisites

Before deploying the platform, ensure you have the following prerequisites in place.

### OpenShift Cluster Requirements

Order an instance of **Red Hat Openshift AI 3.0** from Red Hat Demo Platform with the largest instance size available (128GB of RAM). This cluster will have OpenShift AI 3.0 preconfigured.

### Command-Line Tools

Install the following tools on your local machine:

- **oc CLI** (link available in your OpenShift cluster's **?** help menu)
- **Ansible 2.9 or later**
- **Git**

### GitHub Requirements

You'll need:
- A GitHub Organization (you can create one for free)
- Admin access to create a GitHub App in that organization
- Ability to fork repositories to your organization

### API Keys

> [!NOTE]
> OpenAI is used for _existing_ services deployed in the demo. OpenShift AI inference is used for _new_ services you deploy as part of the demo.

- **OpenAI API key** (or compatible LLM endpoint)
  - Get from [OpenAI Platform](https://platform.openai.com/api-keys)
  - Alternative: Use vLLM or any OpenAI-compatible endpoint
  - Store securely - you'll add this to the Ansible inventory file used to setup services

## GitHub Organization Setup

### Step 1: Create or Use a GitHub Organization

**Why you need an organization:**
- Red Hat Developer Hub authentication (for this demo) requires a GitHub organization
- The demo will create new repositories in your organization using a Developer Hub (Backstage) Template
- Backstage entities and templates will be loaded from your forked repository

**Create a new organization:**

1. Go to [GitHub Organizations](https://github.com/settings/organizations)
2. Click **New organization**
3. Choose a plan (Free is sufficient for testing)
4. Enter organization name (e.g., `acme-corp`, `my-company-demo`)
5. Add billing email
6. Select "My personal account" for ownership
7. Click **Create organization**

**Or use an existing organization** where you have admin access.

### Step 2: Fork This Repository

**IMPORTANT:** You must fork this repository to your organization before deployment.

1. Navigate to https://github.com/evanshortiss/from-ai-hype-to-platform-reality
2. Click **Fork** in the top-right corner
3. Select your organization as the owner
4. Keep the repository name: `from-ai-hype-to-platform-reality`
5. Click **Create fork**

**Why fork?**
- RHDH will load catalog and templates from YOUR fork
- GitHub App will create new repositories in YOUR organization
- You can customize templates and configuration for your needs

Your forked repository will be at:
```
https://github.com/YOUR-ORG/from-ai-hype-to-platform-reality
```

## GitHub App Setup

The platform uses a GitHub App for:
- User authentication in Red Hat Developer Hub (members of your org can login to Developer Hub)
- Creating GitHub repositories from templates
- Managing pull requests, webhooks, and other integrations, e.g showing GitHub Actions data in Developer Hub

### Step 3: Create a GitHub App

> [!IMPORTANT]
> Create the GitHub App in the SAME organization where you forked the repository.

1. Navigate to your GitHub organization settings (the org where you forked the repository)
2. Go to **Developer settings** → **GitHub Apps** → **New GitHub App**

3. **Configure Basic Information:**
   - **GitHub App name**: `Red Hat Developer Hub` (or your preferred name)
   - **Homepage URL**: `https://backstage-developer-hub-ai-rhdh.apps.YOUR-CLUSTER-DOMAIN.com` (you'll update this after deployment)
   - **Webhook**: Uncheck "Active" for now

4. **Configure Permissions:**

   **Repository permissions:** (10 permissions required)
   - **Actions**: Read
   - **Administration**: Read & Write
   - **Attestations**: Read
   - **Checks**: Read
   - **Commit statuses**: Read
   - **Contents**: Read & Write
   - **Environments**: Read & Write
   - **Issues**: Read & Write
   - **Pull requests**: Read & Write
   - **Workflows**: Read & Write

   **Organization permissions:**
   - **Members**: Read-only

   **User permissions:**
   - **Email addresses**: Read-only

   **Note:** Some permissions like Environments Read & Write may not be strictly necessary for basic functionality but are recommended for full template capabilities.

6. **Where can this GitHub App be installed?**
   - Select "Only on this account" 

7. Click **Create GitHub App**

### Step 4: Generate Client Secret

After creating the app:

1. Scroll to **Client secrets** section
2. Click **Generate a new client secret**
3. **Copy the secret immediately** - you won't be able to see it again
4. Store it securely (you'll need it for the Ansible inventory file)

### Step 5: Generate Private Key

1. Scroll to **Private keys** section
2. Click **Generate a private key**
3. A `.pem` file will download automatically
4. Save this file securely in your deployment directory (e.g., `deployment/github-app-private-key.pem`)
5. Do NOT commit this file to git

### Step 6: Note Your Credentials

You'll need these values for the inventory configuration:

- **App ID**: Found at the top of the GitHub App settings page
- **Client ID**: Found in the **About** section
- **Client Secret**: The secret you generated in Step 2
- **Private Key File**: Path to the `.pem` file from Step 5

### Step 7: Install the App

1. Click **Install App** in the left sidebar
2. Select your organization
3. Choose either:
   - **All repositories** (recommended for demo)
   - **Only select repositories** (for production)
4. Click **Install**

## OpenShift Cluster Preparation

### Authenticate to OpenShift

```bash
# Login to your cluster
oc login --server=https://api.YOUR-CLUSTER-DOMAIN.com:6443 --username=admin

# Verify cluster access
oc whoami
oc cluster-info

# Verify you have cluster-admin role
oc auth can-i create namespace
# Should return "yes"
```

## Clone Your Forked Repository

After forking the repository to your organization (see GitHub Organization Setup above):

```bash
# Clone YOUR forked repository (replace YOUR-ORG with your organization name)
git clone https://github.com/YOUR-ORG/from-ai-hype-to-platform-reality.git
cd from-ai-hype-to-platform-reality
```

**Example:**
```bash
# If your organization is "acme-corp"
git clone https://github.com/acme-corp/from-ai-hype-to-platform-reality.git
cd from-ai-hype-to-platform-reality
```

## Next Steps

Once all prerequisites are in place, proceed to the [Deployment Guide](02-deployment-guide.md) to configure and deploy the platform.