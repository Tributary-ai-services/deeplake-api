# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Tributary AI Services for DeepLake** is a production-ready, universal vector database service built with Deep Lake, providing both HTTP REST and gRPC APIs for vector storage, search, and management. It serves as the centralized vector database for the TAS platform, enabling semantic search, embedding storage, and AI-powered document intelligence.

## Data Models & Schema Reference

### Service-Specific Data Models
This service's data models are comprehensively documented in the centralized data models repository:

**Location**: `../aether-shared/data-models/deeplake-api/`

#### Key Vector Database Models:
- **Vector Structure** (`vector-structure.md`) - 13-field schema including embeddings, metadata, timestamps, and source tracking
- **Dataset Organization** (`dataset-organization.md`) - Multi-tenant dataset architecture with tenant isolation
- **Embedding Models** (`embedding-models.md`) - Supported embedding models (OpenAI, Sentence Transformers) and dimension configurations
- **Query API** (`query-api.md`) - Vector similarity search, text search, hybrid search, and filtering capabilities

#### Cross-Service Integration:
- **Document Upload Flow** (`../aether-shared/data-models/cross-service/flows/document-upload.md`) - Vector embedding generation and storage workflow
- **Platform ERD** (`../aether-shared/data-models/cross-service/diagrams/platform-erd.md`) - Complete entity relationship diagram
- **ID Mapping Chain** (`../aether-shared/data-models/cross-service/mappings/id-mapping-chain.md`) - Cross-service identifier relationships

#### When to Reference Data Models:
1. Before making schema changes to vector structure or adding new tensor fields
2. When implementing new embedding models or changing dimension configurations
3. When debugging search accuracy issues or query performance problems
4. When onboarding new developers to understand the vector database architecture
5. Before modifying dataset organization or tenant isolation features

**Main Documentation Hub**: `../aether-shared/data-models/README.md` - Complete navigation for all 38 data model files

## Technology Stack

- **Language**: Python 3.9+
- **Framework**: FastAPI (HTTP REST) + gRPC
- **Vector Database**: Deep Lake
- **Storage**: MinIO (S3-compatible for dataset persistence)
- **Authentication**: JWT tokens with Keycloak integration
- **Monitoring**: Prometheus metrics + Grafana dashboards

## Key Features

### Dual API Support
- **HTTP REST API**: RESTful endpoints for vector operations
- **gRPC API**: High-performance RPC interface for service-to-service communication

### Production Ready
- Authentication and authorization with JWT tokens
- Rate limiting and request throttling
- Comprehensive metrics and monitoring
- Health checks (liveness and readiness probes)

### Multi-Tenant Architecture
- Secure tenant isolation at the dataset level
- Resource management and quota enforcement
- Per-tenant configuration and embedding models

### High Performance
- Optimized for large-scale vector operations
- Batch vector insertion and updates
- Efficient similarity search with filtering
- Support for millions of vectors per tenant

## Common Commands

See comprehensive documentation in:
- `README.md` - Platform overview and quick start
- `DEVELOPMENT.md` - Development setup and workflows
- `MONITORING_ACCESS.md` - Monitoring and observability
- `SECURITY.md` - Security best practices

## API Endpoints

### Vector Operations
- `POST /datasets/{dataset_name}/vectors` - Insert vectors
- `POST /datasets/{dataset_name}/search` - Vector similarity search
- `GET /datasets/{dataset_name}/vectors/{vector_id}` - Retrieve vector
- `PUT /datasets/{dataset_name}/vectors/{vector_id}` - Update vector
- `DELETE /datasets/{dataset_name}/vectors/{vector_id}` - Delete vector

### Dataset Management
- `POST /datasets` - Create dataset
- `GET /datasets` - List datasets
- `GET /datasets/{dataset_name}` - Get dataset info
- `DELETE /datasets/{dataset_name}` - Delete dataset

### Search Capabilities
- **Vector Search**: Cosine similarity search with k-nearest neighbors
- **Text Search**: Full-text search on document content
- **Hybrid Search**: Combined vector and text search
- **Filtered Search**: Metadata-based filtering with search

## Integration Points

- **AudiModal**: Document embedding generation and vector storage
- **Aether Backend**: Semantic search for notebooks and documents
- **TAS LLM Router**: Context retrieval for RAG (Retrieval-Augmented Generation)
- **MinIO**: Persistent storage for Deep Lake datasets
- **Keycloak**: Multi-tenant authentication and authorization

## Important Notes

- Vector dimensions must match the configured embedding model (default: 1536 for OpenAI Ada-002)
- Tenant isolation is critical - always verify tenant context in API requests
- Dataset names follow pattern: `{tenant_id}_{dataset_type}` for proper isolation
- Embedding models are configurable per tenant for specialized use cases
- Integration with shared TAS infrastructure via `tas-shared-network` Docker network
