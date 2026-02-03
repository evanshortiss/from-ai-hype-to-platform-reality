# Backstage Catalog Entities

This directory contains Backstage entity definitions for the demo. These entities are imported into Red Hat Developer Hub to provide a comprehensive Software Catalog.

## Overview

The catalog defines the complete platform architecture in Backstage format:
- **1 Domain** - Business domain (Customer Service)
- **1 System** - Logical system grouping (AI Customer Service Platform)
- **11 Components** - Microservices
- **10 Resources** - Kafka topics
- **Multiple APIs** - Service interfaces
- **Infrastructure Resources** - Kafka, Milvus, Model Registry

## Catalog Structure

### catalog-info.yaml

Root catalog entity that serves as the entry point for Backstage to discover all other entities.

```yaml
kind: Location
spec.type: url
```

This file imports all other entity definitions via relative references.

### domain.yaml

Defines the business domain this platform serves:

- **Domain**: `customer-service`
- **Description**: Customer Service Operations
- **Owner**: Platform Team

### system.yaml

Defines the logical system containing all components:

- **System**: `ai-customer-service-platform`
- **Description**: AI-Powered Customer Service Platform
- **Domain**: `customer-service`
- **Owner**: Platform Team

### resources.yaml

Defines infrastructure resources:

- **Kafka Cluster** - Message broker
- **Milvus Vector Database** - Semantic search
- **OpenAI API** - LLM provider

These are shared resources used by multiple components.

### kafka-topics.yaml

Defines all Kafka topics used for service communication. Each topic is defined as a Backstage Resource entity with type `topic`.

### apis.yaml

Defines API interfaces exposed by services:

- REST APIs for external integration
- Internal service APIs
- Model Context Protocol (MCP) APIs

### services/

Individual component definitions for all microservices:

#### Core Processing Services
- **moderation.yaml** - Content moderation
- **structured-output.yaml** - JSON extraction
- **router-ai.yaml** - Intelligent routing

#### AI-Powered Assistants
- **finance-ai.yaml** - Finance agent
- **support-ai.yaml** - Support agent (RAG-enabled)
- **website-ai.yaml** - Website agent

#### Data & API Services
- **customer-data.yaml** - Customer data management
- **finance-api.yaml** - Financial records REST API
- **document-embedding.yaml** - Document embedding and vector storage
- **website-mcp.yaml** - MCP server for website operations

#### User Interface
- **customer-support-ui.yaml** - Customer support UI

Each service definition includes:
- Component metadata
- Owner team
- System membership
- API definitions
- Dependencies on resources and other components
- Links to repository, CI/CD, and monitoring

## Team Ownership

Entities are organized by team ownership:

- **Platform Engineering**: Infrastructure, RHDH, Kafka, router service
- **Safety Team**: Content moderation
- **AI Team**: Structured output, document embedding
- **Finance Team**: Finance agent, finance API
- **Support Team**: Support agent, customer support UI
- **Website Team**: Website agent, website MCP server
- **Customer Team**: Customer data management

Each entity has an `owner` field referencing the responsible team.

## Registering the Catalog in Backstage

The Ansible deployment automatically configures Red Hat Developer Hub to load this catalog. The configuration is in:

```
deployment/roles/ocp4_workload_ai_summit2025/templates/rhdh/rhdh-app-config.yaml.j2
```

Manual registration URL:
```
https://github.com/evanshortiss/from-ai-hype-to-platform-reality/blob/main/catalog/catalog-info.yaml
```

### Viewing in Developer Hub

After deployment:

1. Access Red Hat Developer Hub
2. Navigate to **Catalog**
3. Filter by:
   - **Domain**: customer-service
   - **System**: ai-customer-service-platform
   - **Owner**: Select a team
   - **Kind**: Component, Resource, API

## Using the Catalog

### Discovering Services

Browse all services in the Catalog view:
- View service metadata and documentation
- See deployment status via ArgoCD integration
- Check runtime status via Kubernetes integration
- Follow links to repositories and CI/CD

### Understanding Dependencies

Each component lists its dependencies:
- **dependsOn**: Other components this service depends on
- **consumesApis**: APIs this service consumes
- **providesApis**: APIs this service provides

Example from `support-ai.yaml`:
```yaml
spec:
  dependsOn:
    - resource:default/milvus-vector-db
    - resource:default/kafka-cluster
    - component:default/document-embedding
    - component:default/openai-embedding
```

### Creating New Services

Use the LangChain Agent Python template:
1. Navigate to **Create** in Developer Hub
2. Select template
3. Fill in service details
4. New service is automatically registered in catalog

## Customizing the Catalog

### Adding a New Service

1. Create a new YAML file in `services/` directory:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: my-new-service
  title: My New Service
  description: Description of the service
  annotations:
    github.com/project-slug: evanshortiss/my-new-service
spec:
  type: service
  lifecycle: production
  owner: my-team
  system: ai-customer-service-platform
  dependsOn:
    - resource:default/kafka-cluster
```

2. Reference it in `catalog-info.yaml`:

```yaml
spec:
  targets:
    - ./services/my-new-service.yaml
```

3. Commit and push to trigger catalog refresh

### Adding a New Kafka Topic

Add to `kafka-topics.yaml`:

```yaml
---
apiVersion: backstage.io/v1alpha1
kind: Resource
metadata:
  name: my-new-topic
  title: My New Topic
  description: Description of the topic
  annotations:
    kafka.io/topic-name: my-new-topic
spec:
  type: topic
  owner: platform-team
  system: ai-customer-service-platform
```

## Best Practices

1. **Keep entities synchronized** - Ensure catalog definitions match actual deployed services
2. **Use descriptive names** - Clear titles and descriptions help discoverability
3. **Define dependencies** - Accurate dependency graphs help understand the system
4. **Assign owners** - Every entity should have a clear owner team
5. **Add annotations** - Use annotations for tool integrations (GitHub, Kubernetes, etc.)
6. **Document APIs** - Include OpenAPI specs or AsyncAPI specs where applicable

## Catalog Refresh

Developer Hub automatically refreshes the catalog:
- **Frequency**: Every 5 minutes (configurable)
- **Initial Delay**: 15 seconds after startup
- **Timeout**: 3 minutes

Manual refresh via RHDH UI:
- Navigate to entity page
- Click "Refresh" button

## Validation

Validate entity YAML files before committing:

```bash
# Using Backstage CLI (if available)
backstage-cli catalog validate ./catalog/

# Or use a YAML linter
yamllint catalog/*.yaml catalog/services/*.yaml
```

## Troubleshooting

### Entity Not Appearing in Catalog

1. Check YAML syntax:
   ```bash
   yamllint catalog-info.yaml
   ```

2. Verify entity is referenced in `catalog-info.yaml`

3. Check RHDH logs:
   ```bash
   oc logs -n ai-rhdh deployment/backstage-developer-hub
   ```

4. Trigger manual refresh in RHDH UI

### Broken Dependencies

If dependency references are broken:
- Verify referenced entity exists
- Check entity naming (namespace:kind/name format)
- Ensure referenced entity is imported into catalog

### GitHub Integration Issues

If GitHub links aren't working:
- Verify `github.com/project-slug` annotation is correct
- Check GitHub App has access to repositories
- Ensure repositories exist

## Additional Resources

- [Backstage System Model](https://backstage.io/docs/features/software-catalog/system-model)
- [Catalog Entity Format](https://backstage.io/docs/features/software-catalog/descriptor-format)
- [Red Hat Developer Hub Documentation](https://developers.redhat.com/rhdh)
- [Architecture Overview](../docs/04-architecture-overview.md)
