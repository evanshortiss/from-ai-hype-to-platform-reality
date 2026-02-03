# LangChain AI Agent Python Template

This Backstage Software Template enables self-service creation of AI-powered agents for the Red Hat One demonstration platform. The template generates a complete Python-based LangChain agent that monitors Kafka topics, analyzes messages, and integrates with the platform ecosystem.

## Overview

This template provides:
- **Complete agent implementation** - Python service with LangChain integration
- **Kubernetes manifests** - Deployment, Service, ConfigMap resources
- **ArgoCD configuration** - GitOps-based deployment
- **Backstage integration** - Automatic catalog registration
- **GitHub repository** - Automated repository creation with all necessary files

## What Gets Created

When you use this template in Red Hat Developer Hub, it creates:

1. **GitHub Repository** - New repository under your organization with:
   - Python LangChain agent implementation
   - Dockerfile for containerization
   - Kubernetes manifests in `manifests/` directory
   - ArgoCD Application definition in `manifests/argocd/`
   - catalog-info.yaml for Backstage registration
   - README with service documentation

2. **Catalog Entity** - Automatically registered in RHDH catalog with:
   - Component metadata
   - Owner team assignment
   - System membership
   - Dependencies on Kafka topics and AI models
   - Links to repository and deployment

3. **ArgoCD Application** - Deployed to OpenShift via GitOps:
   - Syncs from the created GitHub repository
   - Deploys to `parasol` or `parasol-ai` namespace
   - Monitors manifests/argocd/ directory
   - Self-healing enabled

## Template Parameters

### Service Information

- **Name** (required)
  - Unique name for the AI agent service
  - Must be lowercase letters, numbers, and hyphens only
  - Example: `analysis-agent`, `routing-monitor`

- **Description** (required)
  - Brief description of what this agent monitors and analyzes
  - Example: "Monitors failed routing messages and sends notifications"

- **Owner** (required)
  - Owner team or user for this agent
  - Selected from catalog using OwnerPicker
  - Must be a Group or User entity in the catalog

- **System** (required)
  - The system this agent belongs to
  - Selected from catalog using EntityPicker
  - Typically: `ai-customer-service-platform`

### Kafka Configuration

- **Consumer Group ID** (required)
  - Kafka consumer group ID for this agent
  - Must be unique across all consumers
  - Must be lowercase letters, numbers, and hyphens only
  - Example: `analysis-agent-group`

- **Monitored Topic** (required)
  - Kafka topic to monitor for messages
  - Selected from catalog (Resource entities with type: topic)
  - Can also enter a custom topic name
  - Example: `routing-failure`

### AI Configuration

- **AI Model** (required)
  - AI model resource to use for analysis
  - Selected from catalog (Component entities with type: model-server)
  - Can also enter a custom model reference
  - Example: `llm-model-server`

### GitHub Configuration

- **GitHub Organization** (required)
  - GitHub organization or username where the repository will be created
  - Must match the organization where you have the GitHub App installed
  - Example: `acme-corp`, `my-company`

**Note:** If you don't want to enter this every time, you can hardcode your organization in the template.yaml file. Edit `template.yaml` and change the `githubOrganization` parameter to have a default value:

```yaml
githubOrganization:
  title: GitHub Organization
  type: string
  default: your-org-name  # Add this line with your org
  description: GitHub organization or username where the repository will be created
```

Then users won't need to fill it in (but can override if needed).

## Generated Service Architecture

### Service Structure

```
<service-name>/
├── src/
│   ├── agent.py              # LangChain agent implementation
│   ├── kafka_consumer.py     # Kafka integration
│   └── config.py            # Configuration management
├── manifests/
│   ├── deployment.yaml       # Kubernetes Deployment
│   ├── service.yaml         # Kubernetes Service
│   ├── configmap.yaml       # Configuration
│   └── argocd/
│       └── application.yaml  # ArgoCD Application
├── Dockerfile               # Container image definition
├── requirements.txt         # Python dependencies
├── catalog-info.yaml        # Backstage entity
└── README.md               # Service documentation
```

### Agent Capabilities

The generated agent includes:

- **Kafka Integration** - Consumer that reads from specified topic
- **LangChain Framework** - AI orchestration and prompt management
- **LLM Integration** - Connects to configured model server
- **Error Handling** - Robust error handling and retry logic
- **Logging** - Structured logging for observability
- **Health Checks** - Kubernetes liveness and readiness probes

### Environment Configuration

The service is configured via environment variables:

- `KAFKA_BOOTSTRAP_SERVERS` - Kafka cluster connection
- `KAFKA_TOPIC` - Topic to monitor (from template parameter)
- `KAFKA_CONSUMER_GROUP` - Consumer group ID (from template parameter)
- `LLM_API_URL` - Model server endpoint (from AI model entity)
- `LLM_MODEL_NAME` - Model name (from AI model entity)

## Using the Template

### In Red Hat Developer Hub

1. **Navigate to Create Page**
   - Click **Create** in the left sidebar
   - Find **LangChain AI Agent for Kafka Message Analysis**

2. **Fill Service Information**
   - Enter unique service name
   - Provide description
   - Select owner team from dropdown
   - Select system (ai-customer-service-platform)

3. **Configure Kafka**
   - Enter unique consumer group ID
   - Select or enter Kafka topic to monitor

4. **Configure AI**
   - Select AI model from catalog
   - Or enter custom model reference

5. **Review and Create**
   - Review all parameters
   - Click **Create**
   - Watch the progress as the template:
     - Fetches template files
     - Populates variables
     - Creates GitHub repository
     - Registers in catalog
     - Creates ArgoCD application

6. **Access Your Service**
   - Follow the provided links to:
     - GitHub repository
     - Catalog entity
     - ArgoCD application
   - Wait for ArgoCD to sync and deploy

### Template URL

The template is registered in RHDH at:
```
https://github.com/evanshortiss/from-ai-hype-to-platform-reality/blob/main/templates/langchain-agent-python/template.yaml
```

This is automatically configured during platform deployment.

## Customizing the Template

### Modifying the Skeleton

The `skeleton/` directory contains the template files that get populated:

- Edit `skeleton/src/agent.py` to change agent logic
- Modify `skeleton/manifests/` to adjust Kubernetes resources
- Update `skeleton/requirements.txt` to add Python dependencies
- Customize `skeleton/catalog-info.yaml` to change catalog metadata

### Variables Available

In skeleton files, use these variables:

- `${{ values.name }}` - Service name
- `${{ values.description }}` - Service description
- `${{ values.owner }}` - Owner team
- `${{ values.system }}` - System reference
- `${{ values.consumerGroup }}` - Kafka consumer group
- `${{ values.monitoredTopic }}` - Kafka topic entity reference
- `${{ values.kafkaTopicName }}` - Actual Kafka topic name
- `${{ values.aiModel }}` - AI model entity reference
- `${{ values.inferenceServerUrl }}` - Model server URL

### Advanced Customization

Modify `template.yaml` to:

- Add new parameters to the form
- Change validation rules
- Add new template steps
- Customize repository settings
- Modify ArgoCD configuration

## Integration with Platform

### Catalog Integration

Created services automatically integrate with the catalog:

- **Appears in Catalog** - Listed under Components
- **Shows Dependencies** - Links to Kafka topics and AI models
- **Team Ownership** - Associated with owner team
- **System Membership** - Part of ai-customer-service-platform

### ArgoCD Integration

Services are deployed via GitOps:

- **Automatic Sync** - ArgoCD watches the GitHub repository
- **Self-Healing** - Automatically corrects drift
- **Rollback** - Easy rollback via Git revert
- **Status in RHDH** - Deployment status visible in catalog

### Kubernetes Integration

Services run on OpenShift:

- **Namespace**: `parasol` or `parasol-ai` (depending on configuration)
- **Labels**: Consistent labeling for organization
- **Resource Limits**: CPU and memory limits configured
- **Health Checks**: Liveness and readiness probes

## Example Use Cases

### 1. Failed Routing Monitor

Monitor the `routing-failure` topic and send notifications:

```yaml
Name: routing-failure-monitor
Description: Monitors failed routing attempts and alerts teams
Monitored Topic: routing-failure
Consumer Group: routing-failure-monitor-group
```

### 2. Message Volume Analyzer

Analyze message volume patterns:

```yaml
Name: volume-analyzer
Description: Analyzes message volume trends across topics
Monitored Topic: router-input
Consumer Group: volume-analyzer-group
```

### 3. Quality Assurance Agent

Monitor messages for quality issues:

```yaml
Name: qa-monitor
Description: Monitors messages for quality and compliance issues
Monitored Topic: moderation-input
Consumer Group: qa-monitor-group
```

## Development Workflow

After creating a service from the template:

1. **Clone Repository**
   ```bash
   git clone https://github.com/evanshortiss/<service-name>.git
   ```

2. **Make Changes**
   - Edit source code
   - Update manifests
   - Test locally (optional)

3. **Commit and Push**
   ```bash
   git add .
   git commit -m "Update agent logic"
   git push
   ```

4. **ArgoCD Syncs Automatically**
   - Changes detected in repository
   - Application synced to cluster
   - New version deployed

## Troubleshooting

### Template Creation Fails

**Issue**: Template execution fails

**Solutions**:
- Verify all required fields are filled
- Check that owner and system exist in catalog
- Ensure GitHub App has repository creation permissions
- Check RHDH logs for detailed error messages

### Service Not Deploying

**Issue**: ArgoCD application not syncing

**Solutions**:
- Check ArgoCD application status in UI
- Verify manifests/ directory structure in repository
- Check for YAML syntax errors in manifests
- Ensure target namespace exists (`parasol` or `parasol-ai`)

### Kafka Connection Errors

**Issue**: Service can't connect to Kafka

**Solutions**:
- Verify Kafka cluster is running
- Check topic exists in `kafka` namespace (use AMQ Streams Console)
- Verify service account permissions
- Check network policies

## Best Practices

1. **Unique Names** - Use descriptive, unique service names
2. **Clear Descriptions** - Help others understand what the agent does
3. **Appropriate Owner** - Assign to the team responsible for maintaining
4. **Right Topic** - Monitor the topic relevant to your use case
5. **Resource Limits** - Set appropriate CPU/memory limits
6. **Structured Logging** - Use structured logging for observability
7. **Error Handling** - Implement robust error handling
8. **Documentation** - Keep README updated with service details

## Additional Resources

- [LangChain Documentation](https://python.langchain.com/)
- [Backstage Templates](https://backstage.io/docs/features/software-templates/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Architecture Overview](../../docs/04-architecture-overview.md)
