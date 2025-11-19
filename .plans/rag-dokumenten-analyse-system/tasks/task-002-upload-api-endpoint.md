# Task-002: Upload-API-Endpoint

## Metadata
- **ID**: task-002
- **Status**: pending
- **Priority**: must
- **Estimate**: 5 Story Points
- **Labels**: backend, api, python, fastapi
- **Assignee**: java-developer
- **Created**: 2025-11-18
- **Updated**: 2025-11-18

## Description

Implementiere den Backend-API-Endpoint für Dokumenten-Upload. Der Endpoint soll Multipart-File-Uploads verarbeiten, Validierung durchführen, Dateien in Object-Storage speichern und asynchrone Processing-Jobs triggern.

**Technical Requirement**: FR-1 (Dokumenten-Upload) aus PRD.

## Acceptance Criteria

- [ ] POST /api/documents Endpoint implementiert
- [ ] Multipart-Form-Data-Parsing (FastAPI File-Upload)
- [ ] Format-Validierung (PDF, DOCX)
- [ ] Größen-Validierung (max 20MB)
- [ ] File-Storage-Integration (S3/MinIO)
- [ ] Unique-Filename-Generation (UUID)
- [ ] Metadata-Speicherung in PostgreSQL (filename, size, user_id, status)
- [ ] Processing-Job in Queue einreihen (Celery-Task)
- [ ] Success-Response < 10 Sekunden für 5MB-Datei
- [ ] Error-Handling (Invalid-Format, Size-Exceeded, Storage-Failure)
- [ ] Rate-Limiting (max 5 Uploads/Minute/User)
- [ ] Virus-Scan-Integration (ClamAV - optional für MVP)

## Dependencies

- **Requires**:
  - Storage-Service konfiguriert (S3/MinIO)
  - PostgreSQL-Schema für documents-Tabelle
  - Celery-Queue eingerichtet
- **Blocks**:
  - task-001 (Frontend benötigt funktionalen API-Endpoint)
  - task-007 (Text-Extraktion benötigt gespeicherte Files)

## Agent Recommendation

**Recommended Agent**: `java-developer`

**Rationale**: Obwohl Backend in Python/FastAPI implementiert wird, hat der Java-Developer-Agent Expertise in Enterprise-Backend-Patterns, API-Design, und robuster Error-Handling - übertragbar auf Python.

**Alternative**: `python-expert` für idiomatisches Python-Coding.

## Implementation Notes

### Technical Approach

```python
from fastapi import FastAPI, File, UploadFile, HTTPException
from fastapi.responses import JSONResponse
import boto3
from celery import current_app as celery_app

@app.post("/api/documents")
async def upload_document(
    file: UploadFile = File(...),
    current_user: User = Depends(get_current_user)
):
    # 1. Validation
    if file.content_type not in ["application/pdf", "application/vnd.openxmlformats-officedocument.wordprocessingml.document"]:
        raise HTTPException(400, "Invalid file format")

    if file.size > 20 * 1024 * 1024:  # 20MB
        raise HTTPException(400, "File too large")

    # 2. Generate unique filename
    file_id = str(uuid.uuid4())
    s3_key = f"documents/{current_user.id}/{file_id}/{file.filename}"

    # 3. Upload to S3
    s3_client.upload_fileobj(file.file, BUCKET_NAME, s3_key)

    # 4. Save metadata to DB
    doc = Document(
        id=file_id,
        user_id=current_user.id,
        filename=file.filename,
        s3_key=s3_key,
        size=file.size,
        status="uploaded"
    )
    db.add(doc)
    db.commit()

    # 5. Trigger processing job
    celery_app.send_task("process_document", args=[file_id])

    return {"id": file_id, "filename": file.filename, "status": "processing"}
```

### Database Schema

```sql
CREATE TABLE documents (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    filename VARCHAR(255) NOT NULL,
    s3_key VARCHAR(512) NOT NULL,
    size INTEGER NOT NULL,
    status VARCHAR(50) DEFAULT 'uploaded',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_documents_user_id ON documents(user_id);
CREATE INDEX idx_documents_status ON documents(status);
```

### Files to Create/Modify
- `backend/app/api/endpoints/documents.py`
- `backend/app/models/document.py`
- `backend/app/schemas/document.py`
- `backend/app/services/storage.py`
- `backend/tests/test_upload_api.py`

## Testing Strategy

### Unit Tests
- Format-Validierung (accept PDF/DOCX, reject .txt/.jpg)
- Größen-Validierung (< 20MB pass, > 20MB fail)
- S3-Upload-Logic (mit Mocking)
- DB-Transaction-Handling
- Error-Cases (S3-Failure, DB-Failure)

### Integration Tests
- End-to-End Upload (File → S3 → DB → Celery-Queue)
- Rate-Limiting-Enforcement
- Authentication-Required (401 wenn nicht eingeloggt)

### Performance Tests
- Upload-Zeit für 5MB-Datei < 10 Sekunden
- Concurrent-Upload-Handling (5 simultane Uploads)

## Notes

**Referenced PRD Sections**:
- FR-1.1 - FR-1.6 (Dokumenten-Upload)
- NFR-1, NFR-2 (Authentication & Row-Level-Security)
- NFR-5 (Encryption in-transit)
- NFR-9 (Rate-Limiting)

**Dependencies**:
- AWS SDK (boto3) oder MinIO-Client
- FastAPI
- SQLAlchemy (ORM)
- Celery (async tasks)
- ClamAV (optional virus-scan)

**Security Considerations**:
- Validate Content-Type header AND file magic bytes (prevent MIME-Type-Spoofing)
- Sanitize filenames (remove path-traversal characters)
- Implement CSRF-Protection
- Log all uploads for audit-trail
