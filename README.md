# Self-hosted AI Starter Kit (Iyeska Fork)

**Self-hosted AI Starter Kit** is an open-source Docker Compose template for local AI and low-code development.

> [!NOTE]
> This is a fork maintained by [Iyeska LLC](https://github.com/guthdx) with customized port allocations to avoid conflicts with other infrastructure services.
>
> Original repository: [n8n-io/self-hosted-ai-starter-kit](https://github.com/n8n-io/self-hosted-ai-starter-kit)

## What's Included

| Service | Port | Description |
|---------|------|-------------|
| **n8n** | 5680 | Low-code workflow automation (400+ integrations) |
| **Qdrant** | 6333 | High-performance vector database |
| **Docling** | 5001 | OCR and document parsing service |
| **Static Files** | 8085 | Nginx file server for extracted images |
| **Ollama** | 11435 | Local LLM inference (optional, uses profile) |
| **PostgreSQL** | internal | n8n database backend |

## Quick Start (Mac with Local Ollama)

```bash
# Clone and configure
git clone https://github.com/guthdx/self-hosted-ai-starter-kit.git
cd self-hosted-ai-starter-kit
cp .env.example .env

# Start services (uses local Ollama at localhost:11434)
docker compose --profile cpu up -d

# Access n8n
open http://localhost:5680
```

The kit is pre-configured to use your local Ollama installation via `host.docker.internal:11434`.

## Port Allocations

This fork uses modified ports to avoid conflicts with other Iyeska infrastructure:

| Service | Original | This Fork | Reason |
|---------|----------|-----------|--------|
| n8n | 5678 | **5680** | Avoid conflict with production n8n |
| Static Files | 8080 | **8085** | Avoid conflict with Traefik |
| Ollama (Docker) | 11434 | **11435** | Avoid conflict with local Ollama |

All container names are prefixed with `ai-starter-` to prevent naming conflicts.

## Installation Options

### For Mac / Apple Silicon (Recommended)

Use your local Ollama for best performance:

```bash
# Install Ollama first: https://ollama.com/
ollama pull llama3.3
ollama pull nomic-embed-text

# Start the kit
docker compose --profile cpu up -d
```

### For Nvidia GPU

```bash
docker compose --profile gpu-nvidia up -d
```

### For AMD GPU (Linux)

```bash
docker compose --profile gpu-amd up -d
```

### CPU Only (includes containerized Ollama)

```bash
# This starts Ollama in Docker at port 11435
docker compose --profile cpu up -d
```

## Included Workflows

### 1. Demo Workflow
Basic chat with Llama 3.3 via local Ollama.
- URL: http://localhost:5680/workflow/srOnR8PAY3u4RSwb

### 2. RAG Document Ingestion Pipeline
Processes documents through Docling, generates embeddings, stores in Qdrant.
- Watches: `/data/shared/rag-files/pending/`
- Processed files move to: `/data/shared/rag-files/processed/`

### 3. RAG Chat Agent
AI agent that answers questions using ingested documents.
- Uses Qdrant vector search for context retrieval
- Responds with document-grounded answers

## Folder Structure

```
shared/
├── rag-files/
│   ├── pending/     # Drop PDFs here for processing
│   └── processed/   # Completed files moved here
├── extracted-images/ # Images extracted by Docling
└── docling-scratch/  # Temporary processing files
```

## Usage

### Processing Documents

1. Activate the "RAG Document Ingestion Pipeline" workflow in n8n
2. Drop PDF files into `shared/rag-files/pending/`
3. Files are automatically:
   - Parsed by Docling (OCR + text extraction)
   - Chunked and embedded with nomic-embed-text
   - Stored in Qdrant vector database
   - Moved to `processed/` folder

### Chatting with Documents

1. Activate the "RAG Chat Agent" workflow
2. Use the n8n Chat interface or the static chat widget at http://localhost:8085
3. Ask questions about your ingested documents

## Environment Variables

Key settings in `.env`:

```bash
# Database
POSTGRES_USER=n8n_user
POSTGRES_PASSWORD=your_secure_password
POSTGRES_DB=n8n

# n8n Security (generate with: openssl rand -hex 32)
N8N_ENCRYPTION_KEY=your_encryption_key
N8N_USER_MANAGEMENT_JWT_SECRET=your_jwt_secret

# Ollama (default uses local Mac Ollama)
# OLLAMA_HOST=host.docker.internal:11434  # Local Mac Ollama
# OLLAMA_HOST=ai-starter-ollama:11434     # Containerized Ollama
```

## Credentials Setup

On first run, configure these credentials in n8n (Settings → Credentials):

### Local Ollama Service
- **Base URL**: `http://host.docker.internal:11434`

### Local Qdrant
- **URL**: `http://ai-starter-qdrant:6333`
- **API Key**: (leave blank for local)

## Upgrading

```bash
docker compose --profile cpu pull
docker compose --profile cpu up -d
```

## Troubleshooting

### Ollama Connection Issues
- Ensure Ollama is running locally: `ollama list`
- Test connection: `curl http://localhost:11434/api/tags`

### n8n Can't Access Files
- Files must be in the `shared/` folder
- Use path `/data/shared/` inside n8n workflows

### Port Conflicts
- Check `PORT_REGISTRY.md` in the parent repository
- Modify ports in `docker-compose.yml` if needed

## Related Documentation

- [n8n Documentation](https://docs.n8n.io/)
- [Ollama Documentation](https://ollama.com/)
- [Qdrant Documentation](https://qdrant.tech/documentation/)
- [Docling Documentation](https://github.com/docling-project/docling-serve)

## License

Apache License 2.0 - see [LICENSE](LICENSE) file.
