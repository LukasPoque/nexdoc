# NexDoc.dev - Detailed Technical Documentation

## System Overview

NexDoc.dev is a sophisticated multi-tenant document creation platform that bridges the gap between traditional document editing and AI-powered content generation. The system enables organizations to collaboratively create reports by combining file resources from multiple sources with intelligent AI assistance powered by Retrieval-Augmented Generation (RAG).

## Architecture Principles

### Microservices Architecture
Components are loosely coupled, communicating through well-defined APIs and message passing. This enables independent scaling, deployment, and maintenance of each service.

### Cloud-Native Design
Built for containerized deployment with automatic scaling, health monitoring, and service discovery.

## Tech Stack Components

### Edge Layer

#### Traefik
A modern reverse proxy and load balancer that serves as the entry point for all HTTP/HTTPS traffic. Traefik provides automatic SSL certificate management via Let's Encrypt, dynamic service discovery, and middleware capabilities. In NexDoc, it integrates with the keycloakopenid middleware to inject authentication headers into all downstream requests.

**Repository:** https://github.com/traefik/traefik

#### keycloakopenid
A Traefik middleware that validates JWT tokens from Keycloak and enriches requests with additional headers. This enables stateless authentication throughout the application stack without requiring each service to implement OAuth2 flows.

**Repository:** https://github.com/Gwojda/keycloakopenid

### Authentication

#### Keycloak
An enterprise-grade identity and access management solution providing OAuth2/OIDC capabilities. 
**Repository:** https://github.com/keycloak/keycloak

### Application Services

#### Frontend (React + Vite)
A modern single-page application built with React and bundled with Vite for optimal development experience and production performance. The frontend provides:
- Responsive UI with material design principles
- Real-time collaborative editing interface with TipTap
- File management and organization tools
- Document preview and export capabilities

#### TipTap Editor
A headless, framework-agnostic rich text editor built on ProseMirror. The team subscription enables:
- Real-time collaborative editing with conflict resolution
- Change tracking and version history
- Custom extensions for report-specific formatting
- Cloud-based synchronization infrastructure

**Repository:** https://github.com/ueberdosis/tiptap

#### Uppy File Manager
A modular file uploader that provides a consistent interface for various file sources. Features include:
- Drag-and-drop file uploads
- Integration with cloud storage providers
- Progress tracking and resumable uploads
- File type validation and preview generation

**Repository:** https://github.com/transloadit/uppy

#### Uppy Companion
A server-side component that securely handles OAuth flows and file transfers from cloud providers. It acts as a proxy, preventing exposure of user credentials to the browser while enabling direct uploads to MinIO.

**Repository:** https://github.com/transloadit/uppy (companion is part of the Uppy monorepo)

#### Backend Service 
A Python FastAPI application that serves as the core application logic layer. It handles:
- RESTful API endpoints for frontend operations
- WebSocket support for real-time updates
- User and organization management
- Space and document lifecycle management
- AI request to R2R instance
- Document lifecycle management
- Export pipeline coordination

#### LiteLLM Gateway
A unified interface for multiple LLM providers that abstracts API differences and provides:
- Request/response logging for audit trails
- Token usage tracking per organization
- Rate limiting and quota management
- Model routing based on task requirements

**Repository:** https://github.com/BerriAI/litellm

#### R2R (RAG Platform)
A comprehensive Retrieval-Augmented Generation platform that provides:

- Document ingestion pipeline with VLLM and parsing
- Vector database for semantic search (PostgreSQL + pgvector)
- Hybrid search combining keyword and semantic matching
- Re-ranking models for result optimization
- Answer generation with citation tracking
- Knowledge graph construction for complex queries
- Agemntic retrieval capabilities and tooluse

### Data Storage

#### PostgreSQL
The primary relational database storing:
- User accounts data
- Organization settings and configurations
- Document metadata and relationships
- Space membership and permissions
- Audit logs and activity tracking
- System configuration and feature flags

**Repository:** https://github.com/postgres/postgres

#### MinIO
An S3-compatible object storage system providing:
- High-performance file storage
- Automatic data redundancy and healing
- Bucket policies for access control
- Event notifications for processing pipelines
- Versioning for document history
- Lifecycle policies for data retention

**Repository:** https://github.com/minio/minio

### Infrastructure

#### K3S
A lightweight Kubernetes distribution that serves as the deployment platform for the entire NexDoc infrastructure. K3S provides:
- **Full Platform Deployment** - All services deployed as Kubernetes workloads
- **Shared Service Architecture** - Single instances of Backend, Keycloak, PostgreSQL, MinIO with application-level multi-tenancy
- **Shared R2R Instance** - Single R2R deployment with application-level multi-tenancy
- **KEDA Autoscaling** - Event-driven scaling for all services
- **Built-in Ingress** - Traefik integration for external traffic routing
- **Persistent Storage** - StatefulSets for databases and object storage
- **Service Discovery** - Native Kubernetes DNS for inter-service communication
- **Resource Management** - Quotas and limits for cost optimization

**Repository:** https://github.com/k3s-io/k3s
**KEDA Repository:** https://github.com/kedacore/keda

#### K3D
A lightweight wrapper around K3S for local Kubernetes development, providing:
- **Local Cluster Management** - Create/destroy K3S clusters for development
- **Multi-node Testing** - Simulate production cluster topology locally
- **Integrated Load Balancer** - Built-in ingress for testing external access
- **Local Registry** - Fast container image distribution during development
- **Volume Mounting** - Direct host path mounting for rapid iteration
- **Port Forwarding** - Easy access to cluster services from localhost

**Repository:** https://github.com/k3d-io/k3d

### Observability (Future)

#### OpenTelemetry
A vendor-neutral observability framework that will provide:
- Distributed tracing across services
- Metrics collection and aggregation
- Structured logging with correlation
- Performance profiling
- Error tracking and alerting

**Repository:** https://github.com/open-telemetry/opentelemetry-collector

#### LGTM Stack
A modern observability platform consisting of:
- **Loki** - Log aggregation and querying
- **Grafana** - Visualization and dashboards
- **Tempo** - Distributed tracing backend
- **Mimir** - Long-term metrics storage

**Repositories:**
- Loki: https://github.com/grafana/loki
- Grafana: https://github.com/grafana/grafana
- Tempo: https://github.com/grafana/tempo
- Mimir: https://github.com/grafana/mimir

#### OpenMeter
Usage-based billing infrastructure that will track:
- LLM token consumption per organization
- Storage usage in MinIO
- API request volumes
- Feature usage analytics
- Cost allocation and invoicing

**Repository:** https://github.com/openmeterio/openmeter

## Data Flow Patterns

### File Management Flow
1. User authenticates via Keycloak, receiving JWT token
2. Traefik validates token and adds organization headers
3. User uploads files through Uppy to Companion
4. Companion streams files directly to MinIO
5. Backend stores file metadata in PostgreSQL
6. Files appear in user's personal file explorer
7. User retains exclusive access to their files initially

### Space Creation and Collaboration Flow
1. User selects files from their file explorer
2. User creates a new Space within the organization
3. Backend creates space records with ownership metadata
4. User can invite other organization members to the space
5. Invited users gain access to all files within the space
6. Backend triggers R2R ingestion for space files
7. All space members can view and work with shared resources

### Document Creation Flow
1. User enters a space and creates new document
2. Document can be created from scratch or using templates
3. TipTap Cloud session initiated by backend
4. All space members can join the editing session
5. Document automatically saved to space storage by backend by exporting
6. Changes tracked with full version history

### AI-Assisted Editing Flow
1. User requests AI assistance within TipTap editor
2. Frontend (via TipTap) sends request to backend with organization context
3. **Multi-tenant Access**: Backend uses X-Organization header to provide organization context to R2R
4. R2R performs retrieval from all files within the space (organization's data with application-level isolation)
6. R2R uses LLMs for search via LiteLLM
7. LLM generates response with citations from space files
8. Backend formats response for editor integration
9. AI suggestions appear as tracked modifications in TipTap
10. Users can accept, reject, or modify AI suggestions

### Export and Access Flow
1. User completes document and requests export
2. Backend retrieves final document from TipTap
3. Document processed into requested format (PDF, DOCX, etc.)
4. Exported file stored in MinIO within the space
5. File immediately appears in space file explorer
6. All space members can access the exported document
7. Users can download files directly from file explorer
8. No signed URLs needed - access controlled by space membership
