# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The **Systemic Language Analysis (SyLA) Pipeline** is a production-ready Python 3.11+ system for automated psychiatric consultation analysis. It processes transcripts to generate clinical insights, risk assessments, and automated SOAP notes using AI/ML technologies, specifically optimized for Brazilian Portuguese and LGPD compliance.

## Architecture

**Event-driven microservices architecture** with:
- **Azure Functions** orchestrator using Durable Functions for workflow coordination
- **4 parallel analysis engines**: Semantic Analysis, Risk Assessment, Narrative Analysis, SOAP Generator  
- **Kubernetes-based scaling** (AKS Python Pods with HPA 2-20 replicas)
- **OpenAI GPT-4o/GPT-4o-Turbo** for AI processing with cost optimization
- **Redis caching** for embeddings and prompt optimization
- **OpenTelemetry** distributed tracing for observability

## Key Technologies

- **Runtime**: Python 3.11+ with `async/await` patterns throughout
- **AI/ML**: OpenAI GPT-4o, spaCy + transformers for NLP, `tiktoken` for cost management
- **Cloud**: Azure Functions (Python v2), AKS, Blob Storage, CosmosDB, Service Bus
- **API**: FastAPI for REST endpoints
- **Monitoring**: OpenTelemetry v1.21+ with comprehensive health checks

## Development Commands

### Initial Setup (Required)
```bash
# Create project structure
mkdir -p syla_orchestrator syla_analysis syla_api syla_security k8s tests config

# Initialize Python package structure
touch syla_analysis/__init__.py syla_api/__init__.py syla_security/__init__.py

# Create virtual environment
python3.11 -m venv venv
source venv/bin/activate

# Install dependencies (after creating requirements.txt)
pip install -r requirements.txt
```

### Testing (Once Implemented)
```bash
# Run all tests
pytest

# Run integration tests specifically
pytest tests/test_integration.py

# Performance tests  
pytest -m performance

# End-to-end pipeline testing
pytest tests/test_integration.py::test_end_to_end_analysis_pipeline

# Test with coverage
pytest --cov=syla_analysis --cov=syla_api
```

### Local Development
```bash
# Run FastAPI development server
uvicorn syla_api.main:app --host 0.0.0.0 --port 8080 --reload

# Run with Docker Compose (local environment)
docker-compose up -d

# API health check
curl -f http://localhost:8080/health
```

### Azure Functions Development
```bash
# Install Azure Functions Core Tools
npm install -g azure-functions-core-tools@4

# Run Azure Functions locally
cd syla_orchestrator && func start
```

### Kubernetes Operations
```bash
# Deploy main application
kubectl apply -f k8s/syla-deployment.yaml

# Deploy Redis cache
kubectl apply -f k8s/redis-deployment.yaml

# Check system health
kubectl get pods --all-namespaces | grep -E "(Error|CrashLoop|Pending)"

# View logs
kubectl logs -f deployment/syla-app
```

### Code Quality & Linting
```bash
# Format code
black syla_analysis/ syla_api/ syla_security/

# Lint code
flake8 syla_analysis/ syla_api/ syla_security/

# Type checking
mypy syla_analysis/ syla_api/ syla_security/

# Security scanning
bandit -r syla_analysis/ syla_api/ syla_security/
```

### Monitoring
```bash
# Check overnight processing metrics
az monitor metrics list --resource $COSMOS_DB_ID --metric "TotalRequests"

# View application logs
az monitor log-analytics query --workspace $LOG_ANALYTICS_WORKSPACE --analytics-query "traces | where customDimensions.component == 'syla-pipeline'"
```

## Current Status

**⚠️ IMPLEMENTATION STATUS: Documentation-Only Repository**

This repository currently contains comprehensive technical documentation but **no actual implementation**. All Python modules, configuration files, and infrastructure components need to be built according to the specifications in `syla_pipeline_python_docs.md`.

## Expected File Structure (To Be Implemented)

Core implementation structure:
```
syla_orchestrator/
├── function_app.py          # Azure Functions orchestrator
├── requirements.txt         # Azure Functions dependencies
└── host.json               # Azure Functions configuration

syla_analysis/
├── __init__.py
├── semantic_analyzer.py     # Semantic analysis engine
├── risk_analyzer.py         # Risk assessment engine  
├── narrative_analyzer.py    # Narrative analysis engine
├── soap_generator.py        # SOAP note generator
├── cost_optimizer.py        # Token cost management
└── prompt_manager.py        # Prompt templates

syla_api/
├── main.py                 # FastAPI REST API
├── models.py               # Pydantic models
├── routes/                 # API route handlers
└── middleware.py           # Auth & logging middleware

syla_security/
├── data_protection.py      # LGPD compliance
├── access_control.py       # JWT authentication
└── audit.py               # Audit logging

k8s/
├── syla-deployment.yaml    # Main application deployment
├── redis-deployment.yaml   # Redis cache deployment
└── ingress.yaml           # Load balancer configuration

tests/
├── test_integration.py     # End-to-end pipeline tests
├── test_semantic.py        # Semantic analysis tests
├── test_risk.py           # Risk assessment tests
└── fixtures/              # Test data

config/
├── production.yaml         # Production environment
├── development.yaml        # Development environment
└── local.yaml             # Local development

Root level files needed:
├── requirements.txt        # Python dependencies
├── pyproject.toml         # Project configuration
├── Dockerfile             # Container build
├── docker-compose.yml     # Local development
└── Makefile              # Build automation
```

## Performance Targets

- **Processing Time**: ~2 minutes for 400k token transcript
- **Quality Score**: >95% correlation with clinical assessment  
- **Cost Efficiency**: ~$0.15 per analysis
- **Availability**: 99.5% SLA with automatic failover
- **Throughput**: 180+ analyses per hour

## Required Environment Variables

- `OPENAI_API_KEY` - OpenAI API access
- `AZURE_SUBSCRIPTION_ID` - Azure subscription 
- `STORAGE_CONNECTION` - Azure Blob Storage
- `SERVICE_BUS_CONNECTION` - Azure Service Bus
- `COSMOS_CONNECTION` - CosmosDB connection
- `KEY_VAULT_URL` - Azure Key Vault

## Security & Compliance

The system is designed for **LGPD compliance** with:
- JWT-based authentication with role-based access control
- Data encryption at rest and in transit
- PII detection and redaction capabilities
- Automated audit trails
- Data residency requirements in Brazil Southeast

## Implementation Guidelines

### Code Standards
- **Python Version**: 3.11+ with full async/await patterns
- **Type Hints**: Required for all function signatures and class definitions
- **Docstrings**: Google-style docstrings for all public methods
- **Error Handling**: Comprehensive exception handling with structured logging
- **Testing**: Minimum 90% code coverage for all analysis modules

### AI/ML Integration Patterns
- **Cost Control**: Token counting with `tiktoken` before every OpenAI API call
- **Prompt Management**: Centralized prompt templates with versioning
- **Model Selection**: GPT-4o for complex analysis, GPT-4o-Turbo for SOAP generation
- **Caching Strategy**: Redis caching for embeddings and repeated prompt patterns
- **Rate Limiting**: Built-in backoff strategies for API reliability

### LGPD Compliance Requirements
- **Data Minimization**: Process only necessary transcript segments
- **Encryption**: AES-256 for data at rest, TLS 1.3 for data in transit
- **Audit Trails**: All data access and processing must be logged
- **Data Residency**: Brazil Southeast region requirement for all storage
- **PII Detection**: Automatic redaction of personal identifiers

### Performance & Reliability
- **Async Processing**: All I/O operations must use async/await
- **Circuit Breakers**: Implement circuit breaker pattern for external dependencies
- **Health Checks**: Comprehensive health endpoints for Kubernetes probes
- **Telemetry**: OpenTelemetry tracing for all major operations
- **Graceful Degradation**: Fallback strategies when AI services are unavailable

### Cost Optimization Patterns (To Be Implemented)
- Token counting and budget management using `tiktoken`
- Intelligent prompt caching with Redis
- Batch processing for similar requests  
- Model selection optimization (GPT-4o vs GPT-4o-Turbo based on task complexity)