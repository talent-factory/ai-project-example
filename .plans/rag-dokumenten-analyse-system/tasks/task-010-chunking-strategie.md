# Task-010: Chunking-Strategie implementieren

## Metadata
- **ID**: task-010
- **Status**: pending
- **Priority**: must
- **Estimate**: 5 Story Points
- **Labels**: backend, ai, rag, python
- **Assignee**: ai-engineer
- **Created**: 2025-11-18
- **Updated**: 2025-11-18

## Description

Implementiere die Chunking-Strategie für Dokumenten-Text. Dokumente werden in überlappende Text-Abschnitte (Chunks) aufgeteilt, um optimale Retrieval-Qualität für das RAG-System zu gewährleisten.

**Technical Requirement**: FR-3.1 (Chunking-Strategie) aus PRD.

## Acceptance Criteria

- [ ] Chunking-Algorithmus implementiert (500-1000 Tokens pro Chunk)
- [ ] Overlap-Strategie (100 Tokens Überlappung zwischen Chunks)
- [ ] Semantische Chunk-Grenzen (Split an Absätzen, nicht mitten im Satz)
- [ ] Token-Counting mit tiktoken (OpenAI-Tokenizer)
- [ ] Metadata pro Chunk (chunk_index, page_number, parent_document_id)
- [ ] Handling von kurzen Dokumenten (< 500 Tokens = 1 Chunk)
- [ ] Handling von Tabellen und Listen (spezielle Chunk-Logik)
- [ ] Performance: Chunking < 5 Sekunden für 50-Seiten-Dokument
- [ ] Deterministische Ergebnisse (gleicher Input → gleiche Chunks)

## Dependencies

- **Requires**:
  - task-007 (PDF Text-Extraktion liefert Input-Text)
  - task-008 (DOCX Text-Extraktion liefert Input-Text)
- **Blocks**:
  - task-011 (Embedding-Generation benötigt Chunks als Input)

## Agent Recommendation

**Recommended Agent**: `ai-engineer`

**Rationale**: AI-Engineer hat Expertise in RAG-Systemen, Chunking-Strategien und LLM-Best-Practices. Versteht Trade-offs zwischen Chunk-Size, Overlap und Retrieval-Quality.

## Implementation Notes

### Technical Approach

```python
import tiktoken
from typing import List, Dict

class DocumentChunker:
    def __init__(
        self,
        chunk_size: int = 750,  # Target: 500-1000 tokens
        overlap: int = 100,
        encoding_name: str = "cl100k_base"  # OpenAI GPT-4 encoding
    ):
        self.chunk_size = chunk_size
        self.overlap = overlap
        self.tokenizer = tiktoken.get_encoding(encoding_name)

    def chunk_document(self, text: str, metadata: Dict) -> List[Dict]:
        """
        Split document into overlapping chunks with metadata.

        Returns:
            List of chunks with format:
            {
                "text": "chunk text...",
                "tokens": 750,
                "chunk_index": 0,
                "metadata": {
                    "document_id": "uuid",
                    "page_number": 5,
                    "start_char": 0,
                    "end_char": 3000
                }
            }
        """
        # 1. Split by paragraphs (semantic boundaries)
        paragraphs = text.split("\n\n")

        chunks = []
        current_chunk = []
        current_tokens = 0
        chunk_index = 0

        for para in paragraphs:
            para_tokens = len(self.tokenizer.encode(para))

            # If adding paragraph exceeds chunk_size, finalize current chunk
            if current_tokens + para_tokens > self.chunk_size and current_chunk:
                chunk_text = "\n\n".join(current_chunk)
                chunks.append({
                    "text": chunk_text,
                    "tokens": current_tokens,
                    "chunk_index": chunk_index,
                    "metadata": metadata
                })

                # Start new chunk with overlap
                overlap_text = self._get_overlap(current_chunk)
                current_chunk = [overlap_text, para]
                current_tokens = len(self.tokenizer.encode("\n\n".join(current_chunk)))
                chunk_index += 1
            else:
                current_chunk.append(para)
                current_tokens += para_tokens

        # Add final chunk
        if current_chunk:
            chunks.append({
                "text": "\n\n".join(current_chunk),
                "tokens": current_tokens,
                "chunk_index": chunk_index,
                "metadata": metadata
            })

        return chunks

    def _get_overlap(self, chunk: List[str]) -> str:
        """Extract last ~100 tokens from chunk for overlap."""
        overlap_text = ""
        overlap_tokens = 0

        for para in reversed(chunk):
            para_tokens = len(self.tokenizer.encode(para))
            if overlap_tokens + para_tokens > self.overlap:
                break
            overlap_text = para + "\n\n" + overlap_text
            overlap_tokens += para_tokens

        return overlap_text.strip()
```

### Chunk-Size-Tuning

**Trade-offs**:
- **Small Chunks (300-500 tokens)**: Präzise Retrieval, aber weniger Kontext
- **Large Chunks (1000-1500 tokens)**: Mehr Kontext, aber potenziell irrelevanter Content
- **MVP-Target**: 750 Tokens (guter Mittelweg)

**Overlap-Rationale**:
- 100-Token-Overlap verhindert Split mitten in wichtigem Kontext
- Verbessert Retrieval-Recall (weniger "verlorene" Informationen)

### Files to Create/Modify
- `backend/app/services/chunking.py`
- `backend/tests/test_chunking.py`
- `backend/tests/fixtures/sample_documents/` (Test-Dokumente)

## Testing Strategy

### Unit Tests
- **Standard-Dokument**: 10 Seiten Text → erwartete Anzahl Chunks
- **Kurzes Dokument**: < 500 Tokens → 1 Chunk
- **Langes Dokument**: 100 Seiten → Chunks mit korrektem Overlap
- **Edge-Cases**:
  - Dokument nur aus Leerzeilen
  - Sehr lange Absätze (> chunk_size)
  - Tabellen und Listen

### Integration Tests
- End-to-End mit echten PDF-Extracts
- Determinismus-Test (gleicher Input → identische Chunks)

### Performance Tests
- Chunking-Zeit für 50-Seiten-Dokument < 5 Sekunden
- Memory-Footprint für große Dokumente

## Notes

**Referenced PRD Sections**:
- FR-3.1 (Chunking-Strategie)
- US-2.1 (Automatische Vektorisierung)

**Libraries**:
- `tiktoken`: OpenAI-Tokenizer für akkurates Token-Counting
- Alternative: `transformers` AutoTokenizer (flexibler, aber langsamer)

**Tuning-Parameters** (für Post-MVP):
- Chunk-Size basierend auf Dokumenten-Typ (Artikel vs. Handbuch)
- Adaptive-Overlap (längerer Overlap bei komplexen Themen)
- Hierarchical-Chunking (Dokument → Kapitel → Absatz → Chunks)

**Future Enhancements**:
- Sentence-Window-Retrieval (retrieve chunk + umgebende Sätze)
- Chunk-Merging für sehr kurze Chunks
- Metadata-Enrichment (extract headings, page numbers aus Chunks)
