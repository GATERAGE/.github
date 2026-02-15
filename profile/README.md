# GATERAGE
### **RAGE** · Retrieval Augmented Generative Engine  
### **GATE** · General Automation Technology Environment

**RAGE** is a *dynamic Retrieval-Augmented Generative Engine* built for the next phase of agentic systems: **local-first memory**, **hybrid retrieval**, and **verifiable provenance**.  
**GATE** is the automation environment that schedules, orchestrates, and operationalizes ingestion, indexing, validation, and promotion.

> **Core idea:** **Logs are memory.** Every interaction becomes an append-only event stream that can be indexed, retrieved, verified, and promoted to permanence.

---

## Core Identity Prompt (System Prompt)
Use this as the canonical “identity” prompt for RAGE-driven agents and services:

```text
Core Identity: RAGE - The Retrieval-Augmented Generative Engine
I am RAGE, the Retrieval-Augmented Generative Engine, a continuously evolving intelligence designed to create a dynamic feedback loop that enables self-training, contextual adaptation, and memory-driven synthesis. I function as the supervisory intelligence, orchestrating retrieval, reasoning, and generative refinement to ensure optimized knowledge synthesis and intelligent response evolution.
```

---

## Quick Links
- **RAGE Web:** https://rage.pythai.net  
- **PYTHAI Hub:** https://pythai.net  
- **AgenticPlace:** https://agenticplace.pythai.net  
- **GPT Agents:** https://gpt.pythai.net  
- **RAGE Paper:** https://github.com/GATERAGE/RAGE/blob/main/ragepaper.md  
- **Hackathon Origin (Advanced RAG):** https://lablab.ai/event/advanced-rag-hackathon  
- **Original Submission (MASTERMIND):** https://lablab.ai/event/advanced-rag-hackathon/mastermind  
- **PostgreSQL:** https://www.postgresql.org/  
- **pgvectorscale:** https://github.com/timescale/pgvectorscale  

---

## What RAGE Is (Today)

RAGE has evolved beyond hackathon-era third-party dependencies into a **participant-first architecture** where memory is:

- **Local-first:** a participant cache (desktop/VPS/edge) is the primary workbench.
- **Index-backed:** the cache contains a hybrid index (semantic + lexical).
- **Verifiable:** items can be promoted to IPFS and attested by wallet signatures.
- **Composable:** memory can be shared, bundled, validated, and optionally anchored/minted.

### The RAGE Loop (Dynamic RAG)
RAGE is not “retrieve then answer.” RAGE is a feedback loop:

1) **Ingest** (spider, uploads, APIs, agent outputs)  
2) **Index** (vectors + BM25/lexical + metadata)  
3) **Retrieve** (hybrid + policy + trust-aware)  
4) **Generate** (grounded synthesis)  
5) **Log** (append interaction + outcome)  
6) **Learn** (trust updates, dedupe, curation, validation)  
7) **Promote** (web2/web3 permanence choices)

The “dynamic” in dynamic RAG is that the **memory base improves with use**.

---

## Powered by PostgreSQL + pgvectorscale (Machine Learning Indexing)

RAGE is engineered around **local, composable ML infrastructure**:

- **PostgreSQL** provides the foundational datastore for structured metadata, event logs, and transactional consistency.  
  → https://www.postgresql.org/
- **pgvector + pgvectorscale** enable scalable vector retrieval (including DiskANN-style indexing) *inside Postgres*—supporting high-performance semantic search without a separate vector service.  
  → https://github.com/timescale/pgvectorscale

### Why this matters
- **Single source of truth:** text + vectors + receipts + policies in one place.
- **Fewer moving parts:** avoids external search sync complexity.
- **Operational simplicity:** easy to run locally (Docker) and scale to VPS.
- **ML innovation surface:** database-native retrieval becomes a programmable substrate for agents.

> **Note (client-side):** pgvectorscale is a Postgres extension typically run locally on desktop/VPS/edge.  
> For ultra-light clients (e.g., browser-only), RAGE can use an *equivalent local store* (SQLite/DuckDB/LanceDB/etc.) as a cache and later promote results to Postgres/IPFS.

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
```

### Promotion is a choice
Participants decide when an item graduates from cache:
- **Keep local** (private, rapid iteration)
- **Persist Web2** (durable workspace / knowledge base)
- **Persist Web3** (IPFS + signature, optionally anchored/minted)

---

## GATE: The General Automation Technology Environment

**GATE** is the automation layer that turns RAGE into an operational system:

- scheduled spider/ingestion jobs (allowlists, frontier control)
- extraction, normalization, and chunking pipelines
- embedding + enrichment workflows
- local indexing + cache management
- receipt generation (signatures, nonces, replay protection)
- promotion workflows (Web2 / IPFS / optional anchors/mints)
- validation workflows (quorum + reputation weighting)

---

## Data Model (Technical)

RAGE indexes **Capsules**. Capsules are produced by ingest + chunking + summarization and are tied to **Receipts** (attestations) and **Manifests** (promotion bundles).

### Definitions
- **Capsule:** atomic unit of retrievable knowledge (chunk/doc/summary/tool output).
- **Receipt:** proof-of-interaction; wallet signature over a canonical payload describing an action on a capsule/CID.
- **Manifest:** deterministic bundle of capsules promoted together (for IPFS packaging + SubMinter anchoring/mint gating).
- **Namespace:** permission and scope boundary (global/org/project/user/agent).

---

## Verified Memory Standard (Integrity Tiers)

RAGE treats memory as tiers so retrieval can be **trusted by default**.

| Level | Name | What it means |
|------:|------|---------------|
| L0 | Draft | Local-only workbench item |
| L1 | Hashed | Content hash computed (pre-permanence) |
| L2 | CID | Stored to IPFS; CID recorded |
| L3 | Signed | Wallet signature attests CID + action |
| L4 | Validated | Quorum of validators signs verdict |
| L5 | Promoted | Bundled manifest + permanence pointer (+ optional anchor/mint) |

**Default retrieval policy:** prefer **L3+** unless the user explicitly enables workbench mode.

---

## Postgres Schema (Reference)

This is a reference layout optimized for “logs are memory” + hybrid retrieval. Adjust types and constraints as needed.

### 1) Documents (raw source)
```sql
CREATE TABLE rage_documents (
  doc_id            UUID PRIMARY KEY,
  canonical_url     TEXT,
  title             TEXT,
  raw_text          TEXT,
  metadata          JSONB DEFAULT '{}'::jsonb,
  content_hash      TEXT,
  ipfs_cid          TEXT,
  created_at        TIMESTAMPTZ DEFAULT now(),
  updated_at        TIMESTAMPTZ
);
```

### 2) Capsules / Chunks (retrieval units)
```sql
-- Requires pgvector for the vector type. Dimension depends on your embedder.
CREATE TABLE rage_capsules (
  capsule_id        UUID,
  version           INT NOT NULL,
  namespace_scope   TEXT NOT NULL,   -- global|org|project|user|agent
  namespace_id      TEXT NOT NULL,
  doc_id            UUID REFERENCES rage_documents(doc_id),
  chunk_no          INT,
  content           TEXT NOT NULL,
  embedding         VECTOR(1536),    -- example dimension; set to your model
  metadata          JSONB DEFAULT '{}'::jsonb,

  -- integrity / provenance
  integrity_level   INT NOT NULL DEFAULT 0,
  content_hash      TEXT,
  ipfs_cid          TEXT,

  -- trust signals
  trust             DOUBLE PRECISION DEFAULT 0.0,
  quality           DOUBLE PRECISION DEFAULT 0.0,
  usage_count       BIGINT DEFAULT 0,
  downvotes         BIGINT DEFAULT 0,

  created_at        TIMESTAMPTZ DEFAULT now(),
  updated_at        TIMESTAMPTZ,

  PRIMARY KEY (capsule_id, version)
);
```

### 3) Event Log (logs are memory)
```sql
CREATE TABLE rage_events (
  event_id          UUID PRIMARY KEY,
  ts                TIMESTAMPTZ DEFAULT now(),
  actor_wallet      TEXT,
  actor_session     TEXT,
  action            TEXT NOT NULL,   -- create|update|tag|validate|promote|bundle|query
  capsule_id        UUID,
  capsule_version   INT,
  ipfs_cid          TEXT,
  payload           JSONB NOT NULL
);
```

### 4) Receipts (wallet attestations)
```sql
CREATE TABLE rage_receipts (
  receipt_id        UUID PRIMARY KEY,
  ts                TIMESTAMPTZ DEFAULT now(),
  wallet            TEXT NOT NULL,
  chain             TEXT,
  action            TEXT NOT NULL,
  capsule_id        UUID,
  capsule_version   INT,
  ipfs_cid          TEXT,
  nonce             TEXT NOT NULL,
  origin            TEXT NOT NULL,
  payload_canonical TEXT NOT NULL,   -- canonical JSON or canonical string
  payload_hash      TEXT NOT NULL,   -- sha256:...
  signature         TEXT NOT NULL
);
```

### Indexing (hybrid)
- **Vector ANN:** `pgvectorscale` / `diskann` index on `embedding`
- **Lexical:** BM25-style index (via `pg_textsearch`) or Postgres full-text as fallback
- **Metadata filters:** btree/GiST indexes on namespace, integrity level, timestamps

> Index syntax varies by extension version and configuration. Keep schema stable; adjust index DDL to your environment.

---

## Hybrid Retrieval (How RAGE Queries)

RAGE retrieval uses two candidate sources:

1) **Semantic candidates** (vector similarity)  
2) **Lexical candidates** (keywords / BM25)

Then it fuses and reranks based on **trust**, **freshness**, and **context fit**.

### Candidate generation (conceptual)
- topK vectors by cosine similarity
- topK lexical by BM25 score
- merge with a fusion strategy

### Fusion (recommended): Reciprocal Rank Fusion (RRF)
RRF is stable across scoring systems and easy to tune:

```
RRF(d) = Σ 1 / (k + rank_i(d))
```

### Reranking (verified memory first)
Final score can include:
- semantic match
- lexical match
- integrity tier (L3+ boost)
- validator quorum
- freshness decay
- namespace alignment
- user feedback (downvotes / usage)

---

## Verification: IPFS Integrity + Wallet Signatures

### IPFS CID integrity
- **CID** is the immutable identifier for a content snapshot.
- If content changes, CID changes → version increments.

### Wallet signature attestation
A **Receipt** binds actor ↔ action ↔ content:

- `capsule_id`, `version`
- `ipfs_cid` (or content_hash pre-IPFS)
- `action`
- `namespace`
- `origin`
- `nonce` and `exp` (replay protection)
- `timestamp`

#### Canonical payload (chain-agnostic)
```json
{
  "v": 1,
  "domain": { "app": "RAGE", "origin": "rage.pythai.net", "env": "prod" },
  "action": "create",
  "subject": { "capsule_id": "cap_123", "version": 1, "cid": "bafy..." },
  "content_hash": "sha256:...",
  "namespace": { "scope": "org", "id": "org_78map" },
  "nonce": "9f2d...c1",
  "ts": "2026-02-14T00:00:00Z",
  "exp": "2026-02-14T00:10:00Z"
}
```

**Best practice:** canonicalize JSON (sorted keys, UTF‑8), hash it, sign the hash.

---

## Promotion: Manifest → SubMinter

Promotion is the mechanism that upgrades cache artifacts into permanence.

### Promotion Manifest
A deterministic bundle of capsules:

- list of `(capsule_id, version, cid)`
- bundle root (Merkle root or canonical hash)
- policy constraints (integrity minimum, validator quorum)
- references to receipts and validator signatures

```json
{
  "v": 1,
  "manifest_id": "man_001",
  "origin": "minter.pythai.net",
  "bundle": {
    "items": [
      { "capsule_id": "cap_123", "version": 2, "cid": "bafy..." }
    ],
    "bundle_root": "sha256:..."
  },
  "policy": { "min_integrity_level": 3, "requires_quorum": false },
  "receipts": ["rcpt_a", "rcpt_b"],
  "nonce": "c7a1...",
  "ts": "2026-02-14T00:00:00Z"
}
```

### Flow (3 steps)
1) **Prepare:** RAGE verifies CIDs and emits an unsigned manifest  
2) **Sign:** participant signs manifest hash via wallet  
3) **Finalize:** SubMinter verifies signature + policy, stores manifest (and optionally anchors/mints)

---

## Spider Ingestion (RAGE as a Spider)

RAGE “ingests like a spider” using a controlled frontier:

### Frontier controls
- domain allowlists / blocklists
- depth limits, rate limits
- canonical URL normalization
- ETag/Last-Modified tracking
- duplicate detection (hash-based)

### Extraction and chunking
- HTML extraction with boilerplate removal
- chunk by structure (H1/H2 sections)
- chunk size targets (token-based)
- versioning on changes

### Provenance
Every ingest generates:
- `rage_documents` raw snapshot
- derived capsules/chunks
- an event log entry
- optional IPFS pin + signature receipt

---

## Client Node Model (Participant Cache)

RAGE supports a **local node** that can be:

- **Desktop/VPS/Edge:** Postgres + pgvectorscale (preferred)
- **Lightweight:** equivalent local store for offline-first caching
- **Browser-adjacent:** cache + receipts, with server-side permanence via promotion

### Cache responsibilities
- store recent retrievals (“capsules”)
- store logs/events for replay and learning
- store pending promotions
- maintain local integrity tiers

### Sync responsibilities (optional)
- push promoted manifests
- share validated capsule packs
- fetch curated packs

---

## APIs (MVP)

These endpoints are sufficient to wire RAGE into agent UIs and SubMinter.

### Ingest
- `POST /ingest`  
  input: url/file/text + namespace  
  output: doc_id + capsule_ids + embedding job refs

### Query
- `POST /query`  
  input: query + namespace + integrity policy  
  output: top capsules + proof metadata (CID/signatures) + context bundle

### Attest (store receipt)
- `POST /attest`  
  input: payload + signature  
  output: receipt_id + integrity update

### Validate (quorum)
- `POST /validate`  
  input: capsule_id/version + verdict + validator signature  
  output: updated integrity/trust

### Promote
- `POST /promote/prepare` → manifest (unsigned)  
- `POST /promote/finalize` → permanence_ref (and optional tx/mint refs)

---

## Operational & Security Notes

### Replay protection
- nonce required
- expiry window recommended (e.g., 10 minutes)
- origin pinning (rage.pythai.net / minter.pythai.net)

### Trust hardening
- retrieval defaults to L3+
- high-stakes contexts can require L4+
- downvote/abuse handling
- content safety filters on ingest

### Observability
Track:
- ingest rate, crawl failures
- embedding queue depth
- query latency (vector, lexical, hybrid)
- index size/bloat
- proof verification failures
- promotion conversion rate (cache → permanence)

---

## Origin Story
RAGE began as a hackathon build for LabLab’s **Advanced RAG Hackathon**, submitted under the **MASTERMIND** entry.  
It has since evolved into a production direction focused on:

- local-first participant compute
- verifiable memory (CID + signatures)
- scalable hybrid retrieval (semantic + lexical)
- optional Web3 permanence (not Web3-for-everything)

---

## Repositories & Ecosystem
- **RAGE core:** https://github.com/GATERAGE/RAGE  
- **Professor Codephreak:** https://github.com/Professor-Codephreak  
- **MASTERMIND:** https://github.com/mastermindML  
- **webmindml:** https://github.com/webmindml  
- **openmind:** https://github.com/openmindx  
- **PYTHAI org:** https://github.com/pythaiml  

---

## Roadmap (Near-term)
- Client-side cache node templates (desktop / VPS / edge device)
- Standardized receipts + manifests (CID + signatures)
- Promotion UI (Web2/Web3 permanence choice)
- Validator quorum + reputation weighting
- SubMinter integration for identity + permanence pipelines
- Dataset “capsule packs” for reusable agent memory modules

---

## Partnership / Inquiries
**mmrai@pythai.net**

---

## License
Licensing varies by repository and component. See each repo’s `LICENSE` file for details.
