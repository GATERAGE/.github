# GATERAGE
### **RAGE** · Retrieval Augmented Generative Engine  
### **GATE** · General Automation Technology Environment

**RAGE** is a *dynamic RAG engine* designed for the next phase of agentic systems: **local-first memory**, **hybrid retrieval**, and **verifiable provenance**.  
**GATE** is the automation environment that schedules, orchestrates, and operationalizes ingestion, indexing, validation, and promotion.

> **Core idea:** **Logs are memory.** Every interaction becomes an append-only event stream that can be indexed, retrieved, verified, and promoted to permanence.

---

## Quick Links
- **RAGE Web:** https://rage.pythai.net  
- **PYTHAI Hub:** https://pythai.net  
- **AgenticPlace:** https://agenticplace.pythai.net  
- **GPT Agents:** https://gpt.pythai.net  
- **RAGE Paper:** https://github.com/GATERAGE/RAGE/blob/main/ragepaper.md  
- **Hackathon Origin (Advanced RAG):** https://lablab.ai/event/advanced-rag-hackathon  
- **Original Submission (MASTERMIND):** https://lablab.ai/event/advanced-rag-hackathon/mastermind

---

## What RAGE Is (Today)

RAGE has evolved beyond hackathon-era third-party dependencies into a **participant-first architecture**:

### Local-first memory + index (participant cache)
Each participant can run a local node (desktop/VPS/edge) with:
- **Cache:** ingest results, interaction logs, embeddings, metadata, and “capsules”
- **Index:** local hybrid retrieval (semantic + lexical)
- **Policies:** integrity thresholds, namespaces, and promotion rules

### Hybrid retrieval (Elastic-class relevance, local)
RAGE retrieval is built from:
- **Semantic search** (vector ANN)
- **Lexical search** (keyword / BM25-style scoring)
- **Hybrid fusion** (rank fusion + trust/freshness re-ranking)

### Blockchain-compatible by design (verifiable memory)
RAGE does not “search onchain.” It makes memory **auditable**:
- **IPFS CID** = integrity proof (content-addressed permanence)
- **Wallet signature** = authorship / attestation of interaction
- Optional **validator signatures** = community trust and quality gating

---

## Powered by PostgreSQL + pgvectorscale (Machine Learning Indexing)

RAGE is engineered around **local, composable ML infrastructure**:
- **PostgreSQL** is the foundational datastore for structured metadata, logs-as-memory, and transactional consistency.  
  → [PostgreSQL](https://www.postgresql.org/)
- **pgvector + pgvectorscale** enable scalable vector retrieval (including DiskANN-style indexing) directly inside Postgres—supporting high-performance semantic search without a separate vector service.  
  → [pgvectorscale](https://github.com/timescale/pgvectorscale)

This stack supports a clean “single source of truth” model:
- one storage plane for text + vectors + receipts + policies  
- fewer moving parts than syncing external search engines  
- rapid iteration, reproducibility, and production resilience

---

## What GATE Does

**GATE** is the execution environment that turns RAGE into an operational system:
- scheduled spider/ingestion jobs (domain allowlists, frontier control)
- extraction, normalization, and chunking pipelines
- embedding + enrichment workflows
- local indexing + cache management
- receipt generation (signatures, nonces, replay protection)
- promotion workflows (Web2 / IPFS / optional anchors/mints)

---

## Architecture: Cache → Choice → Permanence

RAGE keeps **fast work local**, then upgrades only what matters.

```text
Spider / Inputs
  (web, files, APIs, agent outputs)
        │
        ▼
Participant Cache (Logs = Memory)
  - append-only events
  - local embeddings
  - local hybrid index
        │
        ├──────────────► Web2 Durable Storage
        │                 (cheap, fast, versioned)
        │
        └──────────────► Web3 Permanence
                          - IPFS (CID integrity)
                          - Wallet signatures (receipts)
                          - Optional anchor/mint (SubMinter)
