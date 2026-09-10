# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an interactive workshop for Amazon Bedrock Knowledge Bases, covering RAG patterns from basic retrieval through multi-modal, structured, and graph RAG. It consists of Jupyter notebooks with shared Python utilities — no build system, no tests, no CI.

## Development Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run notebooks
jupyter notebook
```

There is no build, lint, or test suite. All code runs interactively in Jupyter notebooks.

## Architecture

### Layout

- `01-rag-concepts/` — Core RAG: KB creation, retrieve & generate, hybrid search, RAGAS evaluation, document-level ingestion
- `02-optimizing-accuracy-retrieved-results/` — Chunking strategies, metadata filtering, query reformulation, re-ranking
- `03-responsible-ai/` — Guardrails contextual grounding
- `04-multi-modal-rag/` — Audio/video RAG (Bedrock Data Automation) and text+image RAG
- `05-structured-rag/` — SQL generation via Redshift-backed KB
- `06-graph-rag/` — Graph RAG with Neptune Analytics
- `synthetic_dataset/` — Sample PDF, audio, video files for demos
- `utils/` — Shared Python modules imported by all notebooks

### Utility Modules

| Module | Purpose |
|--------|---------|
| `utils/knowledge_base.py` | `BedrockKnowledgeBase` class — orchestrates S3, IAM, OpenSearch Serverless, Neptune, KB creation, data sources, ingestion, and cleanup |
| `utils/knowledge_base_operators.py` | Helper functions for displaying results, S3 operations, DLA ingestion, audio/video playback |
| `utils/evaluation.py` | `KnowledgeBasesEvaluations` — LangChain + RAGAS evaluation integration |
| `utils/managed_knowledge_base.py` | `BedrockManagedKnowledgeBase` — Kendra GenAI Index-backed KBs |
| `utils/structured_knowledge_base.py` | `BedrockStructuredKnowledgeBase` — Redshift-backed KBs |

### Notebook Pattern

Every notebook follows this structure:
1. Install deps from `../requirements.txt`, restart kernel
2. Add parent dir to `sys.path` via `Path().resolve().parent`
3. Import from `utils/` and initialize boto3 clients
4. Generate unique suffix (timestamp or region-account) for resource naming
5. Create KB, upload data, run ingestion
6. Query with `retrieve` or `retrieve_and_generate` APIs
7. Clean up all AWS resources

### Key Models

- **Embedding**: `amazon.titan-embed-text-v2:0` (1024 dimensions)
- **Generation**: `anthropic.claude-3-sonnet-20240229-v1:0` or `anthropic.claude-3-haiku-20240307-v1:0`
- **Reranking**: `cohere.rerank-v3-5:0` or `amazon.rerank-v1:0`

### Resource Naming

Resources use the pattern `bedrock-sample-{type}-{suffix}` where suffix is typically `{region}-{account_id}` or a timestamp.

### Vector Stores

- **OpenSearch Serverless** — default for standard vector RAG
- **Neptune Analytics** — for Graph RAG

## AWS Guidance

- Prefer the AWS MCP Server for AWS interactions — it provides sandboxed execution, observability, and audit logging. If unavailable, use the AWS CLI directly.
- Before starting a task, check whether a relevant AWS skill is available. Load the skill with `retrieve_skill` and prefer its guidance over general knowledge.
- When uncertain about specific AWS details (API parameters, permissions, limits, error codes), verify against documentation rather than guessing. State uncertainty explicitly if you cannot confirm.
- When creating infrastructure, prefer infrastructure-as-code (AWS CDK or CloudFormation) over direct CLI commands.
- When working with infrastructure, follow AWS Well-Architected Framework principles.
- Do not use em dashes in AWS resource names or descriptions. Use hyphens instead.

## Secret Safety

- MUST load the `aws-secrets-manager` skill first for any secret, credential, API key, token, or password task. MUST NOT call `secretsmanager get-secret-value` or `batch-get-secret-value`, and MUST NOT hit the Secrets Manager Agent daemon directly. MUST use `{{resolve:secretsmanager:secret-id:SecretString:json-key}}` with `asm-exec` so the secret resolves at runtime without entering context.
