# Task-026: Unit-Tests Backend

## Metadata
- **ID**: task-026
- **Status**: pending
- **Priority**: must
- **Estimate**: 5 Story Points
- **Labels**: backend, testing, python, quality
- **Assignee**: java-developer
- **Created**: 2025-11-18
- **Updated**: 2025-11-18

## Description

Implementiere umfassende Unit-Tests für alle Backend-Komponenten (APIs, Services, Utils) mit > 80% Code-Coverage-Target gemäß PRD-Anforderungen.

**Technical Requirement**: M4.1 (Unit-Tests > 80% Coverage) aus PRD.

## Acceptance Criteria

- [ ] Test-Coverage > 80% (gemessen mit pytest-cov)
- [ ] Upload-API-Tests (format/size validation, success/error cases)
- [ ] Text-Extraction-Tests (PDF/DOCX)
- [ ] Chunking-Service-Tests (verschiedene Input-Sizes, Edge-Cases)
- [ ] Embedding-Generation-Tests (mit Mocking)
- [ ] RAG-Pipeline-Tests (Query → Retrieval → Generation)
- [ ] Authentication-Tests (Token-Validation, Permissions)
- [ ] Database-Tests (CRUD-Operations, Transactions)
- [ ] Fixtures für Test-Daten (Sample-PDFs, DOCX, Queries)
- [ ] CI-Integration (Tests laufen automatisch bei Push)

## Dependencies

- **Requires**:
  - Alle Backend-Tasks implementiert (task-002, task-007, task-010, task-015, etc.)
- **Blocks**: None (kann parallel zu Feature-Development laufen)

## Agent Recommendation

**Recommended Agent**: `java-developer`

**Rationale**: Enterprise-Testing-Patterns, Test-Driven-Development-Expertise, robuste Mocking-Strategien.

## Implementation Notes

### Test-Struktur

```
backend/tests/
├── conftest.py                    # Shared fixtures
├── unit/
│   ├── test_upload_api.py
│   ├── test_text_extraction.py
│   ├── test_chunking.py
│   ├── test_embedding.py
│   ├── test_rag_pipeline.py
│   └── test_auth.py
├── integration/
│   └── test_e2e_flows.py
├── fixtures/
│   ├── sample.pdf
│   ├── sample.docx
│   └── test_queries.json
└── mocks/
    ├── mock_vector_db.py
    ├── mock_llm.py
    └── mock_s3.py
```

### Example Tests

```python
# tests/unit/test_upload_api.py
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_upload_pdf_success(auth_headers, sample_pdf_file):
    """Test successful PDF upload."""
    files = {"file": ("test.pdf", sample_pdf_file, "application/pdf")}

    response = client.post(
        "/api/documents",
        files=files,
        headers=auth_headers
    )

    assert response.status_code == 200
    data = response.json()
    assert "id" in data
    assert data["filename"] == "test.pdf"
    assert data["status"] == "processing"

def test_upload_invalid_format(auth_headers):
    """Test upload rejection for invalid format."""
    files = {"file": ("test.txt", b"Hello", "text/plain")}

    response = client.post(
        "/api/documents",
        files=files,
        headers=auth_headers
    )

    assert response.status_code == 400
    assert "Invalid file format" in response.json()["detail"]

def test_upload_file_too_large(auth_headers):
    """Test upload rejection for oversized file."""
    large_content = b"x" * (21 * 1024 * 1024)  # 21MB
    files = {"file": ("large.pdf", large_content, "application/pdf")}

    response = client.post(
        "/api/documents",
        files=files,
        headers=auth_headers
    )

    assert response.status_code == 400
    assert "File too large" in response.json()["detail"]

# tests/unit/test_rag_pipeline.py
import pytest
from app.services.rag_pipeline import RAGPipeline
from tests.mocks.mock_vector_db import MockVectorDB
from tests.mocks.mock_llm import MockLLM

@pytest.fixture
def rag_pipeline():
    mock_vdb = MockVectorDB()
    mock_llm = MockLLM()
    return RAGPipeline(vector_db=mock_vdb, llm=mock_llm)

def test_rag_query_with_relevant_chunks(rag_pipeline):
    """Test RAG pipeline with relevant chunks."""
    query = "What is the revenue target?"
    user_id = "user-123"

    # Mock returns relevant chunks
    result = rag_pipeline.query(query, user_id)

    assert "answer" in result
    assert len(result["sources"]) > 0
    assert result["confidence"] > 0.7

def test_rag_query_no_relevant_chunks(rag_pipeline):
    """Test RAG pipeline fallback when no relevant chunks."""
    query = "Irrelevant question"
    user_id = "user-123"

    result = rag_pipeline.query(query, user_id)

    assert "Keine passende Antwort" in result["answer"]
    assert len(result["sources"]) == 0
    assert result["confidence"] == 0.0
```

### Mocking-Strategies

**External-API-Mocking**:
- OpenAI-API (Embeddings, LLM): pytest-mock oder responses
- S3-API: moto (AWS-Mocking-Library)
- Vector-DB: Custom-Mock-Class

**Database-Mocking**:
- Test-DB (SQLite in-memory) für schnelle Tests
- Fixtures für DB-State-Setup

## Testing Strategy

### Coverage-Targets
- **Critical-Path**: 100% (Upload, RAG-Pipeline, Auth)
- **Services**: 90%
- **Utils**: 80%
- **Overall**: > 80%

### Test-Categories
- **Happy-Path**: Standard-Use-Cases
- **Edge-Cases**: Empty-Input, Max-Size, Boundary-Values
- **Error-Cases**: Invalid-Input, API-Failures, Timeouts
- **Security**: SQL-Injection, XSS, CSRF

## Notes

**Referenced PRD Sections**:
- M4.1 (Unit-Tests > 80% Coverage)
- NFR-19 (Modular Architecture für Testability)

**Tools**:
- pytest (Test-Framework)
- pytest-cov (Coverage-Reporting)
- pytest-asyncio (Async-Tests)
- pytest-mock (Mocking)
- moto (AWS-Mocking)
- responses (HTTP-Mocking)

**CI-Integration**:
```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run Tests
        run: |
          pytest --cov=app --cov-report=html
          pytest --cov-fail-under=80
```
