# Inventory Configuration Reference

This document provides a complete reference for all variables in the Ansible inventory file used to deploy the Red Hat One platform.

## Inventory File Location

```
deployment/inventories/inventory
```

Create this file by copying the template:

```bash
cp deployment/inventories/inventory.template deployment/inventories/inventory
```

## Variable Categories

- [Operator Installation](#operator-installation)
- [OpenShift Configuration](#openshift-configuration)
- [GitHub Integration](#github-integration)
- [Default LLM Configuration](#default-llm-configuration)
- [Service-Specific Overrides](#service-specific-overrides)

## Operator Installation

Control whether operators are installed by the playbook. Set to `false` if operators are already installed on your cluster.

### openshift_gitops_operator_install

- **Type**: Boolean
- **Default**: `true`
- **Required**: No
- **Description**: Install the OpenShift GitOps (ArgoCD) operator

```ini
openshift_gitops_operator_install=true
```

Set to `false` if OpenShift GitOps is already installed:

```bash
# Check if installed
oc get csv -n openshift-operators | grep gitops
```

### amq_streams_operator_install

- **Type**: Boolean
- **Default**: `true`
- **Required**: No
- **Description**: Install the AMQ Streams (Kafka) operator

```ini
amq_streams_operator_install=true
```

Set to `false` if AMQ Streams is already installed:

```bash
# Check if installed
oc get csv -n openshift-operators | grep amq-streams
```

### rhdh_operator_install

- **Type**: Boolean
- **Default**: `true`
- **Required**: No
- **Description**: Install the Red Hat Developer Hub operator

```ini
rhdh_operator_install=true
```

Set to `false` if RHDH operator is already installed:

```bash
# Check if installed
oc get csv -n openshift-operators | grep rhdh
```

## OpenShift Configuration

### openshift_ingress_secret_certs_secret

- **Type**: String
- **Default**: `letsencrypt-certs`
- **Required**: Yes
- **Description**: Name of the OpenShift ingress certificate secret

```ini
openshift_ingress_secret_certs_secret=letsencrypt-certs
```

**Common values:**
- `letsencrypt-certs` - Most OpenShift clusters
- `cert-manager-ingress-cert` - RHTAP clusters
- Custom name - If using custom certificates

**How to find the correct value:**

```bash
# List secrets in openshift-ingress namespace
oc get secrets -n openshift-ingress

# Look for a secret containing TLS certificates for the ingress controller
oc get secret -n openshift-ingress <secret-name> -o yaml
```

## GitHub Integration

GitHub App configuration for authentication and repository management in Red Hat Developer Hub.

### github_organization

- **Type**: String
- **Required**: Yes
- **Description**: GitHub organization where the catalog and templates are hosted

```ini
github_organization=evanshortiss
```

**Set this to:**
- `evanshortiss` - If using the original consolidated repository
- Your organization name - If you've forked the repository
- Your username - If using a personal account

This variable is used in:
- RHDH catalog and template URLs
- GitHub App integration
- Organization discovery in the catalog

### github_app_id

- **Type**: Integer
- **Required**: Yes
- **Description**: GitHub App ID from the app settings page

```ini
github_app_id=123456
```

**Where to find:**
1. Go to your GitHub App settings
2. The App ID is displayed at the top: "App ID: 123456"

### github_client_id

- **Type**: String
- **Required**: Yes
- **Description**: GitHub OAuth Client ID from the app settings

```ini
github_client_id=Iv1.abc123def456
```

**Where to find:**
1. Go to your GitHub App settings
2. Find the "Client ID" in the "About" section
3. Format: `Iv1.` followed by alphanumeric characters

### github_client_secret

- **Type**: String (sensitive)
- **Required**: Yes
- **Description**: GitHub OAuth Client Secret generated in app settings

```ini
github_client_secret=ghs_your_actual_client_secret_here
```

**How to generate:**
1. Go to your GitHub App settings
2. Scroll to "Client secrets"
3. Click "Generate a new client secret"
4. **Copy immediately** - you won't see it again
5. Format: `ghs_` followed by 40 characters

**Security note:** Never commit this value to git. Keep the inventory file out of version control.

### github_private_key_file

- **Type**: String (file path)
- **Required**: Yes
- **Description**: Path to the GitHub App private key PEM file

```ini
github_private_key_file=github-app-private-key.pem
```

**Accepted formats:**
- Relative path: `github-app-private-key.pem` (relative to deployment/ directory)
- Absolute path: `/home/user/secrets/github-app-key.pem`

**How to obtain:**
1. Go to your GitHub App settings
2. Scroll to "Private keys"
3. Click "Generate a private key"
4. A `.pem` file will download
5. Save it securely and reference the path here

**Security note:** Never commit the PEM file to git. Add it to `.gitignore`.

## Default LLM Configuration

Default configuration applied to all AI services unless overridden.

### default_llm_api_key

- **Type**: String (sensitive)
- **Required**: Yes
- **Description**: API key for the default LLM provider

```ini
default_llm_api_key=sk-your-api-key-here
```

**Supported providers:**
- OpenAI: `sk-...`
- Azure OpenAI: Custom format
- vLLM: May not require a key
- Any OpenAI-compatible endpoint

**Security note:** Never commit API keys to git.

### default_llm_model_name

- **Type**: String
- **Default**: `gpt-4o-mini`
- **Required**: No
- **Description**: Model name for the default LLM

```ini
default_llm_model_name=gpt-4o-mini
```

**Common values:**
- `gpt-4o-mini` - OpenAI GPT-4o mini (recommended for cost)
- `gpt-4o` - OpenAI GPT-4o (higher quality)
- `gpt-3.5-turbo` - OpenAI GPT-3.5 Turbo
- Custom model name for vLLM or other providers

### default_llm_base_url

- **Type**: String (URL)
- **Default**: `https://api.openai.com/v1`
- **Required**: No
- **Description**: Base URL for the default LLM API endpoint

```ini
default_llm_base_url=https://api.openai.com/v1
```

**Common values:**
- `https://api.openai.com/v1` - OpenAI
- `https://YOUR-RESOURCE.openai.azure.com/` - Azure OpenAI
- `http://vllm-service.namespace.svc.cluster.local:8000/v1` - vLLM in-cluster
- Any OpenAI-compatible endpoint

## Service-Specific Overrides

Override LLM configuration for individual services. Leave blank to use default configuration.

### Moderation Service

Handles content safety and compliance checking.

```ini
moderation_llm_api_key=
moderation_llm_model_name=omni-moderation-latest
moderation_llm_base_url=
```

**Variables:**
- `moderation_llm_api_key` - API key override (or use default)
- `moderation_llm_model_name` - Model name override
- `moderation_llm_base_url` - Base URL override

**Use case:** Use a specialized moderation model like OpenAI's moderation endpoint.

### Structured Output Service

Extracts structured JSON from unstructured text.

```ini
structured_output_llm_api_key=
structured_output_llm_model_name=
structured_output_llm_base_url=
```

**Variables:**
- `structured_output_llm_api_key` - API key override
- `structured_output_llm_model_name` - Model name override
- `structured_output_llm_base_url` - Base URL override

**Use case:** Use a model optimized for JSON extraction (e.g., GPT-4o with structured outputs).

### Router Service

Routes customer requests to specialized agents.

```ini
router_llm_api_key=
router_llm_model_name=
router_llm_base_url=
```

**Variables:**
- `router_llm_api_key` - API key override
- `router_llm_model_name` - Model name override
- `router_llm_base_url` - Base URL override

**Use case:** Use a fast, cost-effective model for routing decisions.

### Finance Agent

Handles financial queries and calculations.

```ini
finance_llm_api_key=
finance_llm_model_name=
finance_llm_base_url=
```

**Variables:**
- `finance_llm_api_key` - API key override
- `finance_llm_model_name` - Model name override
- `finance_llm_base_url` - Base URL override

**Use case:** Use a high-quality model for accurate financial calculations.

### Support Agent

Customer support with RAG-based knowledge retrieval.

```ini
support_llm_api_key=
support_llm_model_name=
support_llm_base_url=

support_embedding_api_key=
support_embedding_model_name=text-embedding-3-small
support_embedding_base_url=
```

**Variables:**
- `support_llm_api_key` - API key for LLM override
- `support_llm_model_name` - Model name override
- `support_llm_base_url` - Base URL override
- `support_embedding_api_key` - API key for embeddings (or use LLM key)
- `support_embedding_model_name` - Embedding model name
- `support_embedding_base_url` - Embedding endpoint override

**Use case:**
- Use a high-quality model for support responses
- Use OpenAI's text-embedding-3-small for semantic search

### Website Agent

Handles website navigation and information queries.

```ini
website_llm_api_key=
website_llm_model_name=
website_llm_base_url=
```

**Variables:**
- `website_llm_api_key` - API key override
- `website_llm_model_name` - Model name override
- `website_llm_base_url` - Base URL override

**Use case:** Use a model with good reasoning for navigating complex websites.

### Document Embedding Service

Generates embeddings for document chunks.

```ini
document_embedding_model_name=text-embedding-3-small
document_embedding_api_key=
document_embedding_base_url=
```

**Variables:**
- `document_embedding_model_name` - Embedding model name
- `document_embedding_api_key` - API key override (or use default)
- `document_embedding_base_url` - Base URL override

**Use case:** Generate embeddings for RAG document storage in Milvus.

## Configuration Examples

### Example 1: Basic OpenAI Configuration

```ini
# Operators (install all)
openshift_gitops_operator_install=true
amq_streams_operator_install=true
rhdh_operator_install=true

# OpenShift
openshift_ingress_secret_certs_secret=letsencrypt-certs

# GitHub
github_organization=evanshortiss
github_app_id=123456
github_client_id=Iv1.abc123def456
github_client_secret=ghs_1234567890abcdef1234567890abcdef12345678
github_private_key_file=github-app-private-key.pem

# Default LLM (OpenAI)
default_llm_api_key=sk-proj-1234567890abcdef1234567890abcdef1234567890ab
default_llm_model_name=gpt-4o-mini
default_llm_base_url=https://api.openai.com/v1

# Service overrides (use defaults)
moderation_llm_api_key=
moderation_llm_model_name=
moderation_llm_base_url=

structured_output_llm_api_key=
structured_output_llm_model_name=
structured_output_llm_base_url=

router_llm_api_key=
router_llm_model_name=
router_llm_base_url=

finance_llm_api_key=
finance_llm_model_name=
finance_llm_base_url=

support_llm_api_key=
support_llm_model_name=
support_llm_base_url=
support_embedding_api_key=
support_embedding_model_name=text-embedding-3-small
support_embedding_base_url=

website_llm_api_key=
website_llm_model_name=
website_llm_base_url=

document_embedding_model_name=text-embedding-3-small
document_embedding_api_key=
document_embedding_base_url=
```

### Example 2: Mixed Provider Configuration

```ini
# Default LLM (OpenAI GPT-4o Mini)
default_llm_api_key=sk-proj-openai-key
default_llm_model_name=gpt-4o-mini
default_llm_base_url=https://api.openai.com/v1

# Finance: Use GPT-4o for higher accuracy
finance_llm_api_key=sk-proj-openai-key
finance_llm_model_name=gpt-4o
finance_llm_base_url=https://api.openai.com/v1

# Support: Use Azure OpenAI
support_llm_api_key=azure-api-key
support_llm_model_name=gpt-4
support_llm_base_url=https://my-resource.openai.azure.com/

# Embeddings: Use OpenAI for all services
support_embedding_api_key=sk-proj-openai-key
support_embedding_model_name=text-embedding-3-small
support_embedding_base_url=https://api.openai.com/v1

document_embedding_api_key=sk-proj-openai-key
document_embedding_model_name=text-embedding-3-small
document_embedding_base_url=https://api.openai.com/v1
```

### Example 3: Self-Hosted vLLM

```ini
# Default LLM (vLLM in-cluster)
default_llm_api_key=none
default_llm_model_name=mistralai/Mistral-7B-Instruct-v0.2
default_llm_base_url=http://vllm-service.ai-models.svc.cluster.local:8000/v1

# Embeddings: Use OpenAI (vLLM doesn't provide embeddings)
support_embedding_api_key=sk-proj-openai-key
support_embedding_model_name=text-embedding-3-small
support_embedding_base_url=https://api.openai.com/v1

document_embedding_api_key=sk-proj-openai-key
document_embedding_model_name=text-embedding-3-small
document_embedding_base_url=https://api.openai.com/v1
```

## Best Practices

### Security

1. **Never commit secrets to git**
   - Add `inventories/inventory` to `.gitignore`
   - Store GitHub App private key securely
   - Use environment variables or secret management tools in production

2. **Use separate API keys per environment**
   - Development keys for testing
   - Production keys for live deployments
   - Set spending limits on API keys

3. **Restrict GitHub App permissions**
   - Only grant required permissions
   - Use organization installation, not user installation
   - Review app activity regularly

### Performance

1. **Choose appropriate models for each service**
   - Use smaller models (gpt-4o-mini) for simple tasks
   - Use larger models (gpt-4o) for complex reasoning
   - Use specialized models for specific tasks (moderation, embeddings)

2. **Consider cost optimization**
   - Default to cost-effective models
   - Override only where quality matters
   - Monitor API usage and costs

3. **Use in-cluster vLLM for cost savings**
   - Host open-source models on OpenShift AI
   - Use for high-volume, low-complexity tasks
   - Fall back to external APIs for complex tasks

### Operations

1. **Document your configuration**
   - Add comments explaining custom values
   - Document any non-standard settings
   - Keep a backup of working configurations

2. **Test incrementally**
   - Start with default configuration
   - Add overrides one service at a time
   - Verify each change before proceeding

3. **Monitor and adjust**
   - Track model performance and costs
   - Adjust models based on observed behavior
   - Update as new models become available

## Validation

Before deploying, validate your configuration:

```bash
# Check syntax
ansible-playbook -i inventories/inventory playbooks/ocp4_workload_ai_summit2025.yml --syntax-check

# Verify GitHub App private key exists
ls -lh deployment/github-app-private-key.pem

# Test OpenShift authentication
oc whoami

# Verify API key format (should start with sk- for OpenAI)
grep default_llm_api_key inventories/inventory
```

## Troubleshooting

### Invalid GitHub App Configuration

**Symptom:** RHDH fails to authenticate with GitHub

**Solutions:**
- Verify all GitHub variables are correct
- Check that the private key file exists and is readable
- Ensure the GitHub App is installed to the organization
- Verify all required permissions are granted

### LLM API Errors

**Symptom:** Services fail with API authentication errors

**Solutions:**
- Verify API keys are correct and active
- Check that base URLs are correct (include `/v1` for OpenAI)
- Ensure model names match available models
- Check API rate limits and quotas

### Embedding Service Failures

**Symptom:** Support or document embedding services fail

**Solutions:**
- Verify embedding model names are correct
- Ensure embedding API keys are valid
- Check that embedding base URLs are correct
- Verify embedding models are available on the endpoint

## Additional Resources

- [OpenAI API Documentation](https://platform.openai.com/docs/api-reference)
- [Azure OpenAI Documentation](https://learn.microsoft.com/en-us/azure/ai-services/openai/)
- [vLLM Documentation](https://docs.vllm.ai/)
- [GitHub Apps Documentation](https://docs.github.com/en/apps)
- [Ansible Inventory Documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
