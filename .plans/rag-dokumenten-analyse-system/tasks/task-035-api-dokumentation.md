# Task-035: API-Dokumentation (Swagger/OpenAPI)

## Metadata
- **ID**: task-035
- **Status**: pending
- **Priority**: should
- **Estimate**: 2 Story Points
- **Labels**: documentation, api, backend
- **Assignee**: markdown-syntax-formatter
- **Created**: 2025-11-18
- **Updated**: 2025-11-18

## Description

Erstelle umfassende API-Dokumentation mit Swagger/OpenAPI für alle REST-Endpoints. Auto-generierte Docs via FastAPI mit Custom-Descriptions, Examples und Authentication-Specs.

**Technical Requirement**: NFR-23 (API-Docs via Swagger) aus PRD.

## Acceptance Criteria

- [ ] Swagger-UI verfügbar unter `/docs`
- [ ] Alle Endpoints dokumentiert (Upload, Query, Documents-List, Delete)
- [ ] Request/Response-Schemas mit Examples
- [ ] Authentication-Spec (JWT-Bearer-Token)
- [ ] Error-Responses dokumentiert (400, 401, 404, 429, 500)
- [ ] Try-It-Out-Functionality funktional
- [ ] OpenAPI-Spec exportierbar (JSON/YAML)
- [ ] README mit Quick-Start-Guide

## Dependencies

- **Requires**: Alle API-Endpoints implementiert
- **Blocks**: None (Nice-to-have für Beta)

## Agent Recommendation

**Recommended Agent**: `markdown-syntax-formatter`

**Rationale**: Dokumentations-Expertise, konsistentes Markdown-Formatting, technisches Schreiben.

## Implementation Notes

### FastAPI-Auto-Documentation

```python
from fastapi import FastAPI, File, UploadFile
from pydantic import BaseModel, Field

app = FastAPI(
    title="RAG Document Analysis API",
    description="API for uploading documents and querying them with AI",
    version="1.0.0",
    contact={
        "name": "Product Team",
        "email": "support@example.com"
    },
    license_info={
        "name": "MIT"
    }
)

class DocumentResponse(BaseModel):
    """Response model for document upload."""
    id: str = Field(..., description="Unique document ID")
    filename: str = Field(..., description="Original filename")
    status: str = Field(..., description="Processing status", example="processing")
    size: int = Field(..., description="File size in bytes")

    class Config:
        schema_extra = {
            "example": {
                "id": "550e8400-e29b-41d4-a716-446655440000",
                "filename": "report.pdf",
                "status": "processing",
                "size": 5242880
            }
        }

@app.post(
    "/api/documents",
    response_model=DocumentResponse,
    summary="Upload a document",
    description="""
    Upload a PDF or DOCX document for analysis.

    **Requirements:**
    - File format: PDF or DOCX
    - Max size: 20MB
    - Authentication: Bearer Token required

    **Processing:**
    Document is asynchronously processed:
    1. Text extraction
    2. Chunking (500-1000 tokens)
    3. Embedding generation
    4. Vector storage

    Returns immediately with status 'processing'.
    """,
    responses={
        200: {"description": "Document uploaded successfully"},
        400: {"description": "Invalid file format or size"},
        401: {"description": "Authentication required"},
        429: {"description": "Rate limit exceeded"}
    },
    tags=["Documents"]
)
async def upload_document(
    file: UploadFile = File(..., description="PDF or DOCX file to upload")
):
    pass  # Implementation
```

### README-Quick-Start

```markdown
# RAG Document Analysis API

## Quick Start

### 1. Authentication

Obtain JWT token:
```bash
curl -X POST https://api.example.com/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "your-password"}'
```

Response:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 900
}
```

### 2. Upload Document

```bash
curl -X POST https://api.example.com/api/documents \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "file=@report.pdf"
```

### 3. Query Document

```bash
curl -X POST https://api.example.com/api/query \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query": "What is the revenue target?"}'
```

## API Reference

See [Swagger Docs](https://api.example.com/docs) for interactive documentation.
```

## Testing Strategy

### Documentation-Quality
- [ ] Alle Endpoints haben Descriptions
- [ ] Examples sind korrekt und funktional
- [ ] Try-It-Out funktioniert ohne Errors
- [ ] Authentication-Flow ist klar dokumentiert

## Notes

**Referenced PRD Sections**:
- NFR-23 (API-Documentation via Swagger)

**Tools**:
- FastAPI (Auto-generates OpenAPI spec)
- Swagger-UI (Interactive docs)
- Redoc (Alternative docs-renderer)
