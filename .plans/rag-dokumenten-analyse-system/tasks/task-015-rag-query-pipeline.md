# Task-015: RAG-Query-Pipeline implementieren

## Metadata
- **ID**: task-015
- **Status**: pending
- **Priority**: must
- **Estimate**: 8 Story Points
- **Labels**: backend, ai, rag, llm, python
- **Assignee**: ai-engineer
- **Created**: 2025-11-18
- **Updated**: 2025-11-18

## Description

Implementiere die core RAG (Retrieval-Augmented Generation) Pipeline: Empfange Nutzer-Query, führe Semantic Search über Vector-DB durch, retrieve relevante Chunks, konstruiere Prompt mit Context und generiere Antwort mit LLM inkl. Quellenangaben.

**User Story**: Als Anna möchte ich eine Frage zu meinen Dokumenten stellen, damit ich schnell Antworten erhalte (US-3.1, US-3.2).

## Acceptance Criteria

- [ ] Query-Embedding-Generierung (OpenAI text-embedding-3-small)
- [ ] Semantic Search über Vector-DB (Top-K=5 relevanteste Chunks)
- [ ] Relevanz-Filtering (Score > 0.7)
- [ ] Prompt-Construction mit Retrieved-Context
- [ ] LLM-Integration (OpenAI GPT-4 Turbo oder Claude 3.5 Sonnet)
- [ ] Antwort-Generierung < 5 Sekunden (P95)
- [ ] Quellenangaben-Extraktion (Dokumentname, Chunk-Index, Seitenzahl)
- [ ] Fallback bei keine relevanten Chunks ("Keine passende Antwort gefunden")
- [ ] Context-Window-Management (max 8K Tokens für Prompt)
- [ ] Halluzination-Prevention (System-Prompt: "nur basierend auf Context")
- [ ] Error-Handling (LLM-API-Timeout, Rate-Limit, Invalid-Response)

## Dependencies

- **Requires**:
  - task-011 (Embeddings in Vector-DB gespeichert)
  - task-012 (Vector-DB operational)
  - task-016 (Prompt-Templates definiert)
- **Blocks**:
  - task-018 (Query-API-Endpoint benötigt RAG-Pipeline)
  - task-020 (Chat-UI benötigt funktionierende Antworten)

## Agent Recommendation

**Recommended Agent**: `ai-engineer`

**Rationale**: Core-Kompetenz in RAG-Systemen, LLM-Integration, Prompt-Engineering und Retrieval-Optimierung. Kritischste Komponente für Antwort-Qualität.

## Implementation Notes

### RAG-Pipeline-Architektur

```python
from typing import List, Dict
import openai
from chromadb import Client

class RAGPipeline:
    def __init__(
        self,
        vector_db: Client,
        embedding_model: str = "text-embedding-3-small",
        llm_model: str = "gpt-4-turbo-preview",
        top_k: int = 5,
        relevance_threshold: float = 0.7
    ):
        self.vector_db = vector_db
        self.embedding_model = embedding_model
        self.llm_model = llm_model
        self.top_k = top_k
        self.relevance_threshold = relevance_threshold

    async def query(self, user_query: str, user_id: str) -> Dict:
        """
        Execute RAG pipeline: Query → Retrieval → Generation

        Returns:
            {
                "answer": "Generated answer...",
                "sources": [
                    {"document": "report.pdf", "page": 5, "chunk_index": 3},
                    ...
                ],
                "confidence": 0.85
            }
        """
        # 1. Generate query embedding
        query_embedding = await self._embed_query(user_query)

        # 2. Retrieve relevant chunks from Vector-DB
        retrieved_chunks = await self._retrieve_chunks(
            query_embedding,
            user_id,
            top_k=self.top_k
        )

        # 3. Filter by relevance score
        relevant_chunks = [
            chunk for chunk in retrieved_chunks
            if chunk["score"] >= self.relevance_threshold
        ]

        if not relevant_chunks:
            return {
                "answer": "Keine passende Antwort in den hochgeladenen Dokumenten gefunden.",
                "sources": [],
                "confidence": 0.0
            }

        # 4. Construct prompt with context
        prompt = self._build_prompt(user_query, relevant_chunks)

        # 5. Generate answer with LLM
        answer = await self._generate_answer(prompt)

        # 6. Extract sources
        sources = self._extract_sources(relevant_chunks)

        return {
            "answer": answer,
            "sources": sources,
            "confidence": self._calculate_confidence(relevant_chunks)
        }

    async def _embed_query(self, query: str) -> List[float]:
        """Generate embedding for user query."""
        response = await openai.Embedding.acreate(
            model=self.embedding_model,
            input=query
        )
        return response["data"][0]["embedding"]

    async def _retrieve_chunks(
        self,
        query_embedding: List[float],
        user_id: str,
        top_k: int
    ) -> List[Dict]:
        """Retrieve top-K most similar chunks from Vector-DB."""
        collection = self.vector_db.get_collection("documents")

        results = collection.query(
            query_embeddings=[query_embedding],
            n_results=top_k,
            where={"user_id": user_id}  # Row-level security
        )

        chunks = []
        for i, (doc_id, distance, metadata, text) in enumerate(zip(
            results["ids"][0],
            results["distances"][0],
            results["metadatas"][0],
            results["documents"][0]
        )):
            similarity_score = 1 - distance  # Convert distance to similarity
            chunks.append({
                "text": text,
                "score": similarity_score,
                "metadata": metadata,
                "rank": i + 1
            })

        return chunks

    def _build_prompt(self, query: str, chunks: List[Dict]) -> str:
        """Construct prompt with retrieved context."""
        context = "\n\n---\n\n".join([
            f"[Quelle {i+1}: {chunk['metadata']['document']}, Seite {chunk['metadata']['page']}]\n{chunk['text']}"
            for i, chunk in enumerate(chunks)
        ])

        prompt = f"""Du bist ein hilfreicher Assistent, der Fragen basierend auf bereitgestellten Dokumenten beantwortet.

WICHTIG: Beantworte die Frage AUSSCHLIESSLICH basierend auf den folgenden Dokumenten-Auszügen. Wenn die Antwort nicht in den Dokumenten steht, sage "Die Informationen in den Dokumenten reichen nicht aus, um diese Frage zu beantworten."

Gebe immer die Quellen an (z.B. "Laut [Quelle 1]...").

KONTEXT AUS DOKUMENTEN:
{context}

FRAGE: {query}

ANTWORT:"""

        return prompt

    async def _generate_answer(self, prompt: str) -> str:
        """Generate answer using LLM."""
        response = await openai.ChatCompletion.acreate(
            model=self.llm_model,
            messages=[
                {"role": "system", "content": "Du bist ein präziser Dokumenten-Assistent."},
                {"role": "user", "content": prompt}
            ],
            temperature=0.3,  # Lower temperature for more factual answers
            max_tokens=1000
        )

        return response.choices[0].message.content

    def _extract_sources(self, chunks: List[Dict]) -> List[Dict]:
        """Extract source metadata from chunks."""
        sources = []
        seen = set()

        for chunk in chunks:
            key = (
                chunk["metadata"]["document"],
                chunk["metadata"]["page"],
                chunk["metadata"]["chunk_index"]
            )

            if key not in seen:
                sources.append({
                    "document": chunk["metadata"]["document"],
                    "page": chunk["metadata"]["page"],
                    "chunk_index": chunk["metadata"]["chunk_index"],
                    "relevance_score": chunk["score"]
                })
                seen.add(key)

        return sources

    def _calculate_confidence(self, chunks: List[Dict]) -> float:
        """Calculate confidence score based on top chunk scores."""
        if not chunks:
            return 0.0
        return sum(c["score"] for c in chunks[:3]) / 3  # Avg of top-3
```

### Prompt-Engineering-Strategies

**System-Prompt**:
- Klare Anweisung: "Nur basierend auf Context"
- Quellenangaben-Requirement
- Fallback-Anweisung bei fehlenden Infos

**Temperature-Tuning**:
- 0.1-0.3: Faktentreu, konservativ (für MVP)
- 0.5-0.7: Kreativer, aber potenziell Halluzinationen (vermeiden)

### Files to Create/Modify
- `backend/app/services/rag_pipeline.py`
- `backend/app/prompts/rag_templates.py`
- `backend/tests/test_rag_pipeline.py`
- `backend/tests/fixtures/test_queries.json`

## Testing Strategy

### Unit Tests
- Query-Embedding-Generation
- Chunk-Retrieval mit Mock-Vector-DB
- Prompt-Construction-Logic
- Source-Extraction
- Confidence-Calculation

### Integration Tests
- End-to-End RAG-Pipeline (Query → Answer)
- Multi-Document-Retrieval
- Fallback-Cases (keine relevanten Chunks)
- Error-Handling (LLM-Timeout, API-Error)

### Quality Tests (Benchmark-Testset)
- **50 Frage/Antwort-Paare** aus Sample-Dokumenten
- **Metrics**:
  - Precision: Korrektheit der Antwort
  - Recall: Wurden alle relevanten Infos gefunden?
  - Halluzination-Rate: Wie oft erfindet LLM Infos?
  - Source-Attribution: Sind Quellen korrekt?

### Performance Tests
- Query-Latenz P50 < 3 Sekunden
- Query-Latenz P95 < 5 Sekunden
- Concurrent-Queries (10 simultane)

## Notes

**Referenced PRD Sections**:
- FR-4 (RAG-Pipeline)
- US-3.1 (Frage stellen)
- US-3.2 (Quellenangaben)
- NFR (Performance-Targets)

**LLM-Provider-Decision**:
- **OpenAI GPT-4 Turbo**: Best-in-class, etabliert, €0.01/1K tokens (output)
- **Claude 3.5 Sonnet**: Längerer Context-Window (200K), bessere Quellenangaben
- **MVP**: Start mit OpenAI, Abstraction-Layer für späteren Multi-Provider-Support

**Cost-Optimization**:
- Response-Caching für identische Fragen (Redis)
- Prompt-Compression (nur relevanteste Chunks)
- Rate-Limiting (max 10 Queries/User/Minute)

**Critical Success Factors**:
1. **Retrieval-Quality**: Richtige Chunks müssen gefunden werden
2. **Prompt-Engineering**: LLM muss korrekt instruiert werden
3. **Source-Attribution**: Quellenangaben müssen akkurat sein
4. **Fallback-Handling**: Ehrlich zugeben, wenn Antwort nicht in Docs

**Future Enhancements**:
- Hybrid-Search (Keyword + Semantic)
- Re-Ranking der Retrieved-Chunks (Cohere Rerank)
- Multi-Turn-Context (Chat-Historie berücksichtigen)
- Streaming-Responses (Server-Sent-Events)
