# Retrieval-Augmented Generation — RAG (Stage 4 Deep Dive)

> **Companion to:** [AI_ROADMAP.md](./AI_ROADMAP.md) — this file is the full Stage 4 deep-dive.
> **Prerequisites:** [Stage 1](./AI_STAGE1_FOUNDATIONS.md) (embeddings, cosine similarity, context window), [Stage 2](./AI_STAGE2_PROMPT_ENGINEERING.md) (structured outputs, evals), [Stage 3](./AI_STAGE3_DAILY_WORKFLOW.md) (habit of building).
> **Audience:** Software developers building their first production-grade RAG system.
> **Time:** 2 weeks of evenings (8–12 focused hours of reading + labs + the mini-project).
> **Approach:** Build a working "chat with my docs" mini-project. Every section feeds it.

---

## Table of Contents

1. [Why RAG (the real problem it solves)](#1-why-rag-the-real-problem-it-solves)
2. [The mental model: RAG = search + LLM](#2-the-mental-model-rag--search--llm)
3. [The full RAG pipeline](#3-the-full-rag-pipeline)
4. [Ingestion: chunking strategies in depth](#4-ingestion-chunking-strategies-in-depth)
5. [Embedding: model choice and tradeoffs](#5-embedding-model-choice-and-tradeoffs)
6. [Storage: vector databases (pgvector deep-dive + alternatives)](#6-storage-vector-databases-pgvector-deep-dive--alternatives)
7. [Retrieval: top-k, hybrid search, and filters](#7-retrieval-top-k-hybrid-search-and-filters)
8. [Re-ranking: the highest-leverage upgrade](#8-re-ranking-the-highest-leverage-upgrade)
9. [Generation: prompt construction, citations, IDK handling](#9-generation-prompt-construction-citations-idk-handling)
10. [Evaluation: how to know your RAG actually works](#10-evaluation-how-to-know-your-rag-actually-works)
11. [Advanced patterns (when basics aren't enough)](#11-advanced-patterns-when-basics-arent-enough)
12. [Failure modes: a field guide](#12-failure-modes-a-field-guide)
13. [Mini-project: "Chat with my docs" end-to-end](#13-mini-project-chat-with-my-docs-end-to-end)
14. [Hands-on labs](#14-hands-on-labs)
15. [Common anti-patterns](#15-common-anti-patterns)
16. [Interview questions companies actually ask](#16-interview-questions-companies-actually-ask)
17. [Self-check: are you done with Stage 4?](#17-self-check-are-you-done-with-stage-4)

---

## 1. Why RAG (the real problem it solves)

Stage 1 established that LLMs are pattern matchers, not databases. They know nothing after their training cutoff, nothing about your private data, and they hallucinate confidently when they don't know.

**RAG fixes this.** It's the single most-deployed AI architecture in industry today, by a large margin. If you can build a competent RAG system, you can immediately ship features at almost any company.

### The three problems RAG solves

```
   PROBLEM                              RAG FIXES IT BY...

   1. "The model has no idea about        Retrieving relevant chunks from
      our internal data."                  your knowledge base at query time
                                           and stuffing them into context.

   2. "The model's knowledge is             Indexing always-current data.
      stale (training cutoff)."             Re-index when docs change.

   3. "The model hallucinates              Grounding: "Answer using ONLY
      facts confidently."                   these retrieved chunks. Cite them.
                                           Say IDK if absent."
```

### Where RAG sits in the job market

Almost every company with internal documents, customer support, code, or proprietary data is building or has built a RAG system. Job postings:

- **AI Engineer / AI Application Engineer** — "build retrieval systems over our internal docs/data" is on most JDs.
- **Forward-Deployed Engineer** — RAG is the bread and butter, deployed customer-by-customer.
- **Search Engineer** (rebranded in many places) — semantic search is increasingly synonymous with RAG infrastructure.
- **Senior Backend Engineer** — increasingly expected to ship a RAG feature at some point.

Interview-wise: RAG system design is **the** most common AI system design question in 2026. If you can walk through the full pipeline confidently — chunking, embedding, retrieval, re-ranking, generation, eval — you'll do well.

### What you'll be able to do after this stage

- Build a working "chat with my docs" system from scratch.
- Choose chunking, embedding, and storage strategies for a given use case.
- Debug a RAG system that's "returning wrong answers" — and know whether it's a retrieval problem or a generation problem.
- Evaluate RAG quality with a golden set, not vibes.
- Discuss RAG architecture with senior engineers without bluffing.

---

## 2. The mental model: RAG = search + LLM

The single most important reframe: **RAG is not magic. It is search + LLM, glued together.**

```
                       ┌───────────────────────────────┐
                       │      THE RAG EQUATION         │
                       │                               │
                       │   RAG  =  Search engine       │
                       │         + Smart formatter     │
                       │                               │
                       │   (retrieves relevant chunks) │
                       │   (writes a grounded answer)  │
                       └───────────────────────────────┘
```

If you've ever built a search index, you've already built 70% of RAG. The LLM at the end is a thin layer that turns retrieved chunks into a fluent, cited answer.

### What this reframe gives you

1. **Debugging clarity.** When RAG returns a wrong answer, the question is binary: did retrieval fail (got the wrong chunks) or did generation fail (got the right chunks, ignored them)? You can log and inspect each step.
2. **Realistic expectations.** RAG is bottlenecked by retrieval quality. If your search returns garbage, the LLM produces garbage. Don't blame the model.
3. **Engineering humility.** Information retrieval is a 50-year-old field. Most "novel" RAG techniques are recycled IR concepts (BM25, re-ranking, query expansion). Lean on that body of knowledge.

### The two-time-phase split (memorize this)

RAG has two completely distinct phases. Conflating them is the #1 source of confusion.

```
       PHASE 1: INDEXING (offline, slow, runs occasionally)
       ────────────────────────────────────────────────────
                                                          .
       Documents  →  Chunk  →  Embed  →  Store in vector DB
       (PDFs,                                              .
        wiki,                                              .
        code...)                                           .

       PHASE 2: QUERY-TIME (online, fast, runs every user request)
       ───────────────────────────────────────────────────────────
                                                          .
       User question → Embed → Search vector DB → Re-rank
                                                          .
                            → Build prompt with retrieved
                              chunks
                                                          .
                            → LLM generates answer
                                                          .
                            → Return answer + citations
```

When you're designing or debugging RAG, always be explicit: am I working on the indexing pipeline or the query pipeline?

---

## 3. The full RAG pipeline

Here's the complete architecture you're going to build. Every box has a section in this doc.

### ASCII version

```
   ╔══════════════════════════════════════════════════════════════════════╗
   ║                          INDEXING PIPELINE                           ║
   ║                          (runs periodically)                         ║
   ╚══════════════════════════════════════════════════════════════════════╝

   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌─────────┐
   │  Documents   │ →  │  Loaders /   │ →  │   Chunker    │ →  │ Embedder│
   │  (PDF, HTML, │    │  Parsers     │    │ (~500 tokens,│    │  (text  │
   │  Markdown,   │    │  (extract    │    │  ~50 overlap)│    │  → vec) │
   │   code, ...) │    │  clean text) │    │              │    │         │
   └──────────────┘    └──────────────┘    └──────────────┘    └────┬────┘
                                                                    │
                                                                    ▼
                                                     ┌─────────────────────┐
                                                     │   Vector DB         │
                                                     │   (pgvector,        │
                                                     │    Chroma, Pinecone)│
                                                     │   stores:           │
                                                     │     • vector        │
                                                     │     • chunk text    │
                                                     │     • metadata      │
                                                     │       (source, etc) │
                                                     └─────────────────────┘
                                                                    │
                                                                    │
   ╔══════════════════════════════════════════════════════════════════════╗
   ║                       QUERY-TIME PIPELINE                            ║
   ║                       (runs every user request)                      ║
   ╚══════════════════════════════════════════════════════════════════════╝
                                                                    │
   ┌──────────────┐    ┌──────────────┐                             │
   │ User question│ →  │   Embedder   │                             │
   └──────────────┘    │ (same model  │                             │
                       │  as indexing)│                             │
                       └──────┬───────┘                             │
                              │                                     ▼
                              │                          ┌──────────────────┐
                              └────────────────────────► │  Vector search   │
                                                         │  (top-k, e.g. 20)│
                                                         │  + filters       │
                                                         └────────┬─────────┘
                                                                  ▼
                                                       ┌──────────────────┐
                                                       │   Re-ranker      │
                                                       │  (cross-encoder, │
                                                       │   Cohere Rerank) │
                                                       │  top-k → top-N   │
                                                       │  (e.g., 20 → 5)  │
                                                       └────────┬─────────┘
                                                                ▼
                                                 ┌────────────────────────┐
                                                 │  Prompt construction   │
                                                 │  [system + question +  │
                                                 │   top-N chunks +       │
                                                 │   "cite sources"]      │
                                                 └────────┬───────────────┘
                                                          ▼
                                                 ┌────────────────────────┐
                                                 │   LLM generates        │
                                                 │   answer + citations   │
                                                 └────────┬───────────────┘
                                                          ▼
                                                 ┌────────────────────────┐
                                                 │   Return to user       │
                                                 └────────────────────────┘
```

### Mermaid version (for visual rendering in GitHub / Obsidian / Notion)

```mermaid
flowchart TD
    subgraph Indexing["📥 Indexing Pipeline (offline)"]
        A[Documents<br/>PDF, HTML, MD, code] --> B[Loaders /<br/>Parsers]
        B --> C[Chunker<br/>~500 tok, 50 overlap]
        C --> D[Embedder<br/>text → vector]
        D --> E[(Vector DB<br/>pgvector / Chroma)]
    end

    subgraph Query["🔍 Query-Time Pipeline (online)"]
        Q[User question] --> QE[Embedder<br/>same model]
        QE --> VS[Vector search<br/>top-k retrieval]
        E -.indexed vectors.-> VS
        VS --> RR[Re-ranker<br/>top-k → top-N]
        RR --> P[Prompt construction<br/>question + chunks +<br/>cite-sources rules]
        P --> L[LLM]
        L --> R[Answer + citations<br/>to user]
    end

    style Indexing fill:#1f3a5f,stroke:#4a90e2,color:#fff
    style Query fill:#3a5f1f,stroke:#7ab84a,color:#fff
    style E fill:#5f1f3a,stroke:#e24a90,color:#fff
```

Internalize this diagram. Every RAG system in production is some variant of this. The advanced patterns in §11 are upgrades to specific boxes; the skeleton is unchanged.

---

## 4. Ingestion: chunking strategies in depth

Chunking is the single most underrated lever in RAG. A bad chunking strategy will defeat the best embedding model and the best LLM. A good one makes mediocre models look brilliant.

### Why chunk at all?

Two reasons:

1. **Embedding models have an input limit** (usually 8k tokens, sometimes more). Documents are often much longer.
2. **Retrieval should return relevant pieces, not whole books.** If a 200-page manual is one giant chunk, retrieving "it" doesn't help — the LLM still has to find the answer inside 200 pages. Smaller chunks = more precise retrieval.

But also: **chunks that are too small lose context.** A sentence ripped from its surrounding paragraph often means nothing.

So chunking is a tradeoff. Too big → loose retrieval. Too small → lost context. The sweet spot depends on your data.

### The chunking strategies, ranked by sophistication

```
   ┌─────────────────────────────────────────────────────────────┐
   │ 1. FIXED-SIZE CHUNKING (the default; surprisingly good)     │
   │                                                             │
   │    Split into N-token chunks with M-token overlap.          │
   │    Typical: 500 tokens, 50 overlap.                         │
   │                                                             │
   │    [_____chunk 1_____]                                      │
   │              [_____chunk 2_____]                            │
   │                        [_____chunk 3_____]                  │
   │                                                             │
   │    ✅ Simple, fast, no surprises                            │
   │    ❌ Cuts mid-sentence and mid-paragraph                   │
   ├─────────────────────────────────────────────────────────────┤
   │ 2. RECURSIVE CHARACTER SPLITTING (the standard upgrade)     │
   │                                                             │
   │    Try splitting at "\n\n" first; if chunk too big,         │
   │    split at "\n"; then ". "; then " ".                      │
   │    Same target size, but respects natural boundaries.       │
   │                                                             │
   │    ✅ Avoids cutting mid-sentence; chunks more coherent     │
   │    ❌ Slightly more complex; still naive about structure    │
   ├─────────────────────────────────────────────────────────────┤
   │ 3. STRUCTURE-AWARE CHUNKING (use it when you can)           │
   │                                                             │
   │    For Markdown: split on headers (## ###).                 │
   │    For HTML: split on <h1>/<h2>/<section>.                  │
   │    For code: split on function/class boundaries.            │
   │                                                             │
   │    ✅ Chunks align with logical units; metadata-rich        │
   │    ❌ Format-specific; need a loader per format             │
   ├─────────────────────────────────────────────────────────────┤
   │ 4. SEMANTIC CHUNKING (experimental, sometimes worth it)     │
   │                                                             │
   │    Embed sentences. Group adjacent sentences whose          │
   │    embeddings are similar; cut when similarity drops.       │
   │                                                             │
   │    ✅ Topically coherent chunks                             │
   │    ❌ Expensive to compute; results vary by domain          │
   ├─────────────────────────────────────────────────────────────┤
   │ 5. PARENT-CHILD / SMALL-TO-BIG (the production pattern)     │
   │                                                             │
   │    Index SMALL chunks (for precise retrieval).              │
   │    But when retrieving, return the LARGER parent chunk      │
   │    (for richer context to the LLM).                         │
   │                                                             │
   │    ✅ Best of both worlds: precise retrieval + context      │
   │    ❌ More complex indexing; need parent-child links        │
   └─────────────────────────────────────────────────────────────┘
```

### Practical starting points

For a general docs/wiki corpus: **start with recursive character splitting, 500 tokens, 50 overlap.** Works surprisingly well. Tune only if you have an eval set telling you it's failing.

For code: **structure-aware**, splitting on function/class boundaries, with the function/file path as metadata.

For long-form documents (books, manuals): **parent-child** is worth the complexity.

### Code: recursive character splitting

This is what LangChain and LlamaIndex give you by default. Here it is from scratch, ~30 lines:

```python
def recursive_split(text: str, chunk_size: int = 500, chunk_overlap: int = 50,
                    separators: list[str] = None) -> list[str]:
    """
    Split text into chunks <= chunk_size tokens (roughly: ~4 chars per token).
    Tries each separator in order, falling back to next if chunks still too big.
    """
    if separators is None:
        separators = ["\n\n", "\n", ". ", " ", ""]

    chunk_size_chars = chunk_size * 4
    overlap_chars = chunk_overlap * 4

    def _split(text: str, seps: list[str]) -> list[str]:
        if len(text) <= chunk_size_chars:
            return [text]

        sep = seps[0]
        if sep == "":
            return [text[i:i + chunk_size_chars]
                    for i in range(0, len(text), chunk_size_chars - overlap_chars)]

        parts = text.split(sep)
        chunks, current = [], ""
        for part in parts:
            candidate = current + (sep if current else "") + part
            if len(candidate) <= chunk_size_chars:
                current = candidate
            else:
                if current:
                    chunks.append(current)
                if len(part) > chunk_size_chars:
                    chunks.extend(_split(part, seps[1:]))
                    current = ""
                else:
                    current = part
        if current:
            chunks.append(current)
        return _add_overlap(chunks)

    def _add_overlap(chunks: list[str]) -> list[str]:
        if len(chunks) <= 1 or overlap_chars == 0:
            return chunks
        out = [chunks[0]]
        for i in range(1, len(chunks)):
            prev_tail = chunks[i-1][-overlap_chars:]
            out.append(prev_tail + chunks[i])
        return out

    return _split(text, separators)
```

In practice you'd use the LangChain `RecursiveCharacterTextSplitter` or LlamaIndex `SentenceSplitter`. But understanding what's happening is the senior-engineer move.

### Always attach metadata

Every chunk should carry metadata, not just text:

```python
{
    "text": "...the actual chunk content...",
    "metadata": {
        "source": "docs/auth/oauth.md",
        "title": "OAuth 2.0 Setup",
        "section": "Authorization Code Flow",
        "url": "https://internal.docs/oauth#code-flow",
        "last_updated": "2026-04-12",
        "doc_type": "wiki",
        "team": "platform",
    }
}
```

You'll use this metadata for:
- **Filtering** ("only retrieve from docs published after 2025")
- **Citations** ("source: docs/auth/oauth.md, section 'Authorization Code Flow'")
- **Boosting** ("recent docs rank higher than old ones")
- **Permissions** ("user only sees docs their team has access to")

Building RAG without metadata is the #1 thing junior engineers do wrong. Always include it.

---

## 5. Embedding: model choice and tradeoffs

Stage 1 introduced embeddings conceptually. Now you choose which model.

### The choice space in 2026

| Model | Provider | Dimensions | Cost ($/1M tokens) | Notes |
|---|---|---|---|---|
| `text-embedding-3-small` | OpenAI | 1536 (configurable) | ~$0.02 | Cheap, strong baseline. Best default. |
| `text-embedding-3-large` | OpenAI | 3072 (configurable) | ~$0.13 | Higher quality. Use if 3-small underperforms in eval. |
| `voyage-3` / `voyage-3-large` | Voyage AI | 1024 / 1024 | ~$0.06 / $0.18 | Strong for retrieval; multilingual variants. |
| `bge-m3` | BAAI (open source) | 1024 | $0 + hosting | Runs locally; multilingual; supports dense + sparse + multi-vector. Strong open default. |
| `e5-large-v2` / `e5-mistral` | Microsoft / others | 1024 / 4096 | $0 + hosting | Open-source workhorses. |
| Cohere Embed v3 | Cohere | 1024 | ~$0.10 | Multilingual; well-tuned for retrieval. |

### How to choose

Don't agonize. **Default to `text-embedding-3-small`** for English. It's cheap, fast, and competitive with much larger models. Upgrade to `large` or try Voyage/BGE only if your eval set tells you to.

For local/private deployments: `bge-m3` is the strongest open default in 2026.

### Things that matter (in priority order)

1. **Same model for indexing and querying.** If you index with model A and query with model B, similarity is meaningless. Lock the model into your config. Re-index from scratch if you ever switch.
2. **Dimensions vs quality.** Higher dimensions = slightly better recall, much higher storage cost. 1024–1536 is the sweet spot.
3. **Domain-specific tuning.** General models are surprisingly good. Domain-tuned models (legal, medical, code) help only on narrow domains and only after you've measured the gap. Don't reach for them first.
4. **Language coverage.** If you're indexing non-English content, use a multilingual model (`text-embedding-3-*`, Voyage multilingual, BGE-M3, Cohere multilingual).

### The MTEB leaderboard

The **Massive Text Embedding Benchmark (MTEB)** ranks embedding models across many tasks. It's at [huggingface.co/spaces/mteb/leaderboard](https://huggingface.co/spaces/mteb/leaderboard).

**Use it for sanity-checking, not gospel.** Leaderboard winners often don't beat OpenAI's models on real-world retrieval tasks with your data. Build your own eval (§10) and let it decide.

### Embedding code (you've seen this; here it is in indexing context)

```python
from openai import OpenAI

client = OpenAI()

def embed_batch(texts: list[str], model: str = "text-embedding-3-small") -> list[list[float]]:
    """Embed up to 2048 texts in one API call. Much cheaper than one-by-one."""
    response = client.embeddings.create(model=model, input=texts)
    return [item.embedding for item in response.data]

chunks = ["chunk 1 text", "chunk 2 text", "chunk 3 text"]
vectors = embed_batch(chunks)
```

Always batch your embedding calls. Embedding APIs accept hundreds of inputs per request. One-by-one is 100x slower and more expensive.

---

## 6. Storage: vector databases (pgvector deep-dive + alternatives)

You need somewhere to put your vectors and the chunk text + metadata. Welcome to vector databases.

### The choice space

```
   ┌──────────────────────────────────────────────────────────────┐
   │ pgvector (Postgres extension)         ⭐⭐⭐⭐⭐  RECOMMENDED  │
   │   Pros: You probably already run Postgres. Same DB for       │
   │         vectors + your existing relational data. Filters,    │
   │         joins, transactions — for free.                      │
   │   Cons: Slower than dedicated vector DBs at >10M vectors.    │
   │   Use:  Default for backend-folks until proven inadequate.    │
   ├──────────────────────────────────────────────────────────────┤
   │ Chroma                                ⭐⭐⭐   LOCAL FIRST    │
   │   Pros: Lightweight, embedded, great for prototypes.         │
   │   Cons: Not great at scale; sparse production tooling.       │
   │   Use:  Quick local experiments, learning, demos.            │
   ├──────────────────────────────────────────────────────────────┤
   │ Pinecone                              ⭐⭐⭐⭐  MANAGED       │
   │   Pros: Fully managed, scales to billions, fast.             │
   │   Cons: Vendor lock-in, costs at scale, separate from your   │
   │         main DB (consistency headaches).                     │
   │   Use:  When you've outgrown pgvector and want zero-ops.     │
   ├──────────────────────────────────────────────────────────────┤
   │ Weaviate / Qdrant / Milvus            ⭐⭐⭐⭐  OPEN-SOURCE   │
   │   Pros: Self-hostable, scalable, feature-rich.               │
   │   Cons: Extra system to operate; team-skill cost.            │
   │   Use:  Large scale or strong privacy needs.                 │
   ├──────────────────────────────────────────────────────────────┤
   │ Elasticsearch / OpenSearch            ⭐⭐⭐   FULL-TEXT++    │
   │   Pros: Mature, you may already run it for BM25.             │
   │         Vector + keyword search in one place.                │
   │   Cons: Vector performance behind dedicated DBs.             │
   │   Use:  Already running ES, want hybrid search natively.     │
   └──────────────────────────────────────────────────────────────┘
```

### Why pgvector first

For 99% of "build my first RAG" projects, `pgvector` is the right answer:

- You already know SQL.
- You can query vectors and structured data together: `SELECT chunk_text FROM chunks WHERE team='platform' ORDER BY embedding <=> $1 LIMIT 10;`
- ACID transactions, backups, replication — all the boring infra you don't want to rebuild.
- One database to operate.
- Scales to millions of vectors without breaking a sweat.

You graduate to a dedicated vector DB when you have a *demonstrated* need (latency, scale, specific features). Not before.

### pgvector schema and queries

```sql
-- 1. Enable the extension (once per database)
CREATE EXTENSION IF NOT EXISTS vector;

-- 2. Create your chunks table
CREATE TABLE chunks (
    id              BIGSERIAL PRIMARY KEY,
    source          TEXT NOT NULL,
    chunk_text      TEXT NOT NULL,
    metadata        JSONB NOT NULL DEFAULT '{}'::jsonb,
    embedding       VECTOR(1536),   -- match your embedding model's dim
    created_at      TIMESTAMPTZ DEFAULT now()
);

-- 3. Build an approximate nearest neighbor (ANN) index
--    HNSW is the recommended algorithm in pgvector 0.5+
CREATE INDEX chunks_embedding_idx
    ON chunks
    USING hnsw (embedding vector_cosine_ops)
    WITH (m = 16, ef_construction = 64);

-- 4. (Optional) Index your metadata for filtering
CREATE INDEX chunks_metadata_gin_idx ON chunks USING gin (metadata);
```

### The three distance operators in pgvector

```sql
embedding <=> query_vector   -- cosine distance (USE THIS for OpenAI embeddings)
embedding <-> query_vector   -- L2 (Euclidean) distance
embedding <#> query_vector   -- negative inner product
```

Use **cosine distance** (`<=>`) for OpenAI, Anthropic, Voyage, BGE, Cohere — all of these are normalized.

### A typical query

```sql
SELECT
    id,
    chunk_text,
    metadata,
    1 - (embedding <=> $1::vector) AS similarity   -- convert distance to similarity
FROM chunks
WHERE
    metadata->>'team' = 'platform'                 -- filter by metadata
    AND created_at > now() - interval '90 days'    -- freshness filter
ORDER BY
    embedding <=> $1::vector                       -- nearest neighbors
LIMIT 20;
```

This single query is most of what you need. Filters + vector search + ordering — all in standard SQL.

### Python integration

```python
import psycopg
from pgvector.psycopg import register_vector

conn = psycopg.connect("postgresql://...")
register_vector(conn)

def insert_chunk(source: str, text: str, embedding: list[float], metadata: dict):
    with conn.cursor() as cur:
        cur.execute(
            """
            INSERT INTO chunks (source, chunk_text, metadata, embedding)
            VALUES (%s, %s, %s, %s)
            """,
            (source, text, json.dumps(metadata), embedding),
        )
    conn.commit()

def search(query_embedding: list[float], k: int = 20, team: str = None):
    with conn.cursor() as cur:
        cur.execute(
            """
            SELECT chunk_text, metadata, 1 - (embedding <=> %s::vector) AS sim
            FROM chunks
            WHERE %s IS NULL OR metadata->>'team' = %s
            ORDER BY embedding <=> %s::vector
            LIMIT %s
            """,
            (query_embedding, team, team, query_embedding, k),
        )
        return cur.fetchall()
```

That's it. No new system, no new query language, no new ops headache. Just Postgres.

### HNSW vs IVFFlat (the index choice)

pgvector supports two ANN index types:

- **HNSW** (Hierarchical Navigable Small World) — better recall, faster queries, slower to build, more memory. **Use this** in modern deployments.
- **IVFFlat** — older, faster to build, lower memory, lower recall. Default before HNSW landed in pgvector 0.5.

Stick with HNSW. The tradeoffs almost always favor it.

---

## 7. Retrieval: top-k, hybrid search, and filters

You have vectors stored. Now you retrieve.

### The basic retrieval: top-k vector search

```
   ┌──────────────────┐
   │  User question   │
   └────────┬─────────┘
            ▼
   ┌──────────────────┐
   │     Embed        │
   └────────┬─────────┘
            ▼
   ┌──────────────────┐         ┌────────────────────────┐
   │ Cosine similarity│  ◄────  │  Vector DB (all chunks │
   │ vs all vectors   │         │  with their vectors)   │
   └────────┬─────────┘         └────────────────────────┘
            ▼
   ┌──────────────────┐
   │  Top-k chunks    │   ◄── typical k = 20
   └──────────────────┘
```

That's it. For many simple use cases this is enough. For most production cases, it's not — you need to layer on filters, hybrid search, and re-ranking.

### How to choose k

- **Too small** (k=3): risk of missing the relevant chunk entirely.
- **Too large** (k=100): too much noise; the LLM gets distracted by irrelevant chunks; also expensive.
- **Sweet spot**: retrieve k=20–50, then re-rank down to top 3–10 (see §8).

Two-stage retrieval (cast a wide net, then re-rank) is the standard pattern.

### Filters: metadata is your friend

Pure vector search ignores hard constraints. You almost always want filters:

- **Permissions**: "user X can only see docs they have access to."
- **Recency**: "only docs from the last 12 months."
- **Source**: "only docs from the engineering wiki, not the marketing site."
- **Doc type**: "only API reference, not tutorials."

In pgvector this is just `WHERE` clauses. In Pinecone/Weaviate, it's metadata filters in the query API.

**Filter before vector search, not after.** Otherwise you might retrieve top-k vectors and find none match your filter, leaving you with zero results.

### Hybrid search: vector + BM25

Vector search is great at semantic similarity but **bad at exact matches**. If a user asks "What does error code E2049 mean?", a vector search may return semantically related errors but miss the exact code.

**BM25** (a classic IR scoring algorithm based on term frequency) is the opposite: brilliant at exact matches, dumb about semantics.

The solution: run both, combine the rankings.

```
                       HYBRID SEARCH
                       ─────────────
                            │
                            ▼
              ┌─────────────────────────────┐
              │      User question          │
              └──────────────┬──────────────┘
                             │
              ┌──────────────┴──────────────┐
              ▼                             ▼
   ┌──────────────────┐         ┌──────────────────┐
   │  Vector search   │         │   BM25 search    │
   │  (semantic)      │         │   (keyword)      │
   │  → top-k         │         │   → top-k        │
   └────────┬─────────┘         └────────┬─────────┘
            │                            │
            └────────────┬───────────────┘
                         ▼
            ┌────────────────────────────┐
            │  Reciprocal Rank Fusion    │
            │  (combines rankings)       │
            │                            │
            │  score = Σ 1/(60 + rank_i) │
            │      over each retriever   │
            └────────────┬───────────────┘
                         ▼
            ┌────────────────────────────┐
            │  Top-k merged results       │
            └────────────────────────────┘
```

### Reciprocal Rank Fusion (RRF) — the simplest hybrid algorithm

For each chunk, sum `1/(60 + rank)` across each retriever, then re-rank by sum. The constant `60` is a smoothing factor; it's the default everywhere.

```python
def reciprocal_rank_fusion(
    ranked_lists: list[list[str]],   # list of lists of chunk_ids
    k_constant: int = 60,
) -> list[tuple[str, float]]:
    """
    Given multiple ranked lists of chunk IDs, return a single fused ranking.
    """
    scores = {}
    for ranked in ranked_lists:
        for rank, chunk_id in enumerate(ranked, start=1):
            scores[chunk_id] = scores.get(chunk_id, 0) + 1 / (k_constant + rank)
    return sorted(scores.items(), key=lambda kv: kv[1], reverse=True)

# Usage
vector_results = ["c3", "c1", "c7", "c2", "c9"]  # top-k from vector search
bm25_results   = ["c1", "c5", "c3", "c8", "c2"]  # top-k from BM25 search

fused = reciprocal_rank_fusion([vector_results, bm25_results])
# → [("c1", 0.0317), ("c3", 0.0314), ("c2", 0.0156), ...]
```

This single function is the entire algorithm. RRF is one of those rare cases where the simplest possible idea works as well as anything fancier.

### Where does BM25 come from?

You need a keyword search index alongside your vector store. Three options:

1. **Postgres full-text search** (built-in `tsvector` / `tsquery`) — works fine for medium scale.
2. **Elasticsearch / OpenSearch** — production-grade, supports BM25 out of the box.
3. **Library**: `rank_bm25` for in-memory BM25 over your chunks.

For pgvector users, the simplest path is Postgres FTS: index your `chunk_text` with `to_tsvector`, query with `to_tsquery`, and you get keyword search for free.

### When hybrid search matters

- Domains with many proper nouns, codes, IDs, or numeric values (e.g., legal, medical, IT support).
- Mixed query styles ("how do I auth" → semantic vs "OAUTH_INVALID_GRANT" → keyword).
- High-precision domains where missing exact terms is unacceptable.

For chatty queries over chatty content, vector-only is often fine. **Add hybrid search when your eval set tells you to** — not by default.

---

## 8. Re-ranking: the highest-leverage upgrade

If you do exactly one thing beyond basic retrieval, do this.

### The problem with first-stage retrieval

Vector search and BM25 are **fast but coarse**. They retrieve "plausibly relevant" chunks. Many in the top-20 are noise — close in vector space but not actually answering the question.

If you stuff all 20 chunks into the LLM, you (a) waste tokens, (b) confuse the model with irrelevant context, (c) trigger the lost-in-the-middle problem.

### The fix: a second-stage re-ranker

A re-ranker is a **smaller specialized model** that scores each candidate chunk against the query. It's much slower per pair than an embedding lookup — but you only run it on the top-20 candidates, so it's affordable.

```
   ┌──────────────────────────────┐
   │ STAGE 1: First-stage retrieval│
   │ (vector + BM25 + RRF)         │
   │   Output: top-20 candidates   │   ← fast, broad
   └──────────────┬───────────────┘
                  ▼
   ┌──────────────────────────────┐
   │ STAGE 2: Re-ranker             │
   │ Score each (query, chunk) pair │   ← slow, precise
   │ with a cross-encoder model     │
   │   Output: top-5 final chunks   │
   └──────────────┬───────────────┘
                  ▼
        Send top-5 to LLM
```

### Bi-encoder vs cross-encoder (the technical difference)

```
   BI-ENCODER (what embedding models do):
   ─────────────────────────────────────
   query  → [encoder] → vector_q
   chunk  → [encoder] → vector_c
                     ↓
              cosine(q, c)

   • Encodes each independently — chunks can be pre-indexed.
   • Fast at query time: one embedding + a similarity search.
   • Less accurate: no cross-attention between query and chunk.


   CROSS-ENCODER (what re-rankers do):
   ────────────────────────────────────
   (query, chunk) together → [encoder] → score

   • Both inputs go in together — full cross-attention.
   • Cannot pre-index — has to run for every (query, chunk) pair.
   • Slower but much more accurate at "how relevant is this?"
```

That's why re-rankers go in stage 2: they're too slow for stage 1 (millions of chunks), but perfect when you've narrowed down to top-20.

### Re-ranker options in 2026

| Re-ranker | Type | Cost / Setup |
|---|---|---|
| **Cohere Rerank v3** | Managed API | ~$2 / 1k queries; 1 line of code |
| **Voyage Rerank-2** | Managed API | Similar pricing; strong quality |
| **`BAAI/bge-reranker-v2-m3`** | Open-source | Free, runs locally; needs a GPU for speed |
| **`mixedbread-ai/mxbai-rerank-large-v2`** | Open-source | Free; strong quality |
| **LLM-as-reranker** | Use a small LLM | Most flexible, more expensive per query |

### Code: Cohere Rerank

```python
import cohere
co = cohere.Client(api_key="...")

def rerank(query: str, candidates: list[dict], top_n: int = 5) -> list[dict]:
    response = co.rerank(
        model="rerank-english-v3.0",
        query=query,
        documents=[c["text"] for c in candidates],
        top_n=top_n,
    )
    return [candidates[r.index] for r in response.results]

# Usage in your RAG pipeline:
candidates = search_vector(query, k=20)         # stage 1
final = rerank(query, candidates, top_n=5)      # stage 2
```

### Code: open-source re-ranker (local, no API cost)

```python
from sentence_transformers import CrossEncoder

reranker = CrossEncoder("BAAI/bge-reranker-v2-m3")

def rerank_local(query: str, candidates: list[dict], top_n: int = 5) -> list[dict]:
    pairs = [(query, c["text"]) for c in candidates]
    scores = reranker.predict(pairs)
    ranked = sorted(zip(candidates, scores), key=lambda x: x[1], reverse=True)
    return [c for c, _ in ranked[:top_n]]
```

### When re-ranking is worth it

- Almost always for production RAG, if the latency budget allows (re-ranker adds 100–500ms).
- Especially when first-stage retrieval is noisy (many similar but not-quite-right chunks).
- Especially when you have lots of chunks per user query and the LLM context is precious.

The typical lift from adding a re-ranker is **5–15 percentage points in retrieval accuracy.** It's the single biggest non-obvious upgrade in the RAG pipeline.

---

## 9. Generation: prompt construction, citations, IDK handling

You have the right chunks. Now you compose the prompt and let the LLM do its job.

### The standard RAG prompt template

```
SYSTEM:
You are a helpful assistant answering questions based ONLY on the
provided context. Follow these rules strictly:

1. Answer ONLY using information from the <context> blocks below.
2. If the answer is not in the context, reply exactly:
   "I don't have information about that in my knowledge base."
3. Cite your sources at the end of relevant sentences in the form [^N],
   where N is the index of the context block you used.
4. Be concise. Do not include caveats, apologies, or filler.
5. Do not invent facts, URLs, or citations.

<context>
[1] (source: docs/auth/oauth.md)
{chunk_1_text}

[2] (source: docs/auth/jwt.md)
{chunk_2_text}

[3] (source: docs/api/security.md)
{chunk_3_text}
...
</context>

USER:
{the user's question}
```

This template has every important pattern baked in:

- **Constrained to source** (§Stage 2 pattern): "ONLY using information from..."
- **Explicit IDK behavior**: don't make stuff up; say IDK.
- **Citations**: forces the model to reference which chunk it used.
- **Anti-filler instruction**: keeps answers tight.
- **Numbered chunks**: makes citations unambiguous.

### Why the IDK instruction matters most

Without it, the model **will** confabulate. With it, the model has an explicit escape hatch and uses it (usually). This single instruction is the difference between a RAG system that fails gracefully and one that lies confidently.

Test this yourself: ask your RAG a question whose answer is **not** in the corpus. Watch what happens. If the model invents an answer, the prompt is wrong.

### Citation styles

| Style | Looks like | When |
|---|---|---|
| Inline numeric | "OAuth uses authorization codes [^1]." | Most common; clean |
| URL-embedded | "OAuth uses authorization codes ([docs/auth/oauth.md](#))." | When you want clickable links |
| End-of-answer | Answer text, then a "Sources:" section | When citations clutter the answer |
| Schema-based | Use structured outputs with a `citations: [int]` field | Programmatic |

For chat UIs, **schema-based** is cleanest — the model returns `{answer: "...", citations: [1, 3]}` and the UI renders citation widgets.

### Handling multi-turn RAG (the chat case)

When the user asks a follow-up that references previous context — "what about the second one?" — you have a problem: embedding "what about the second one" returns nothing useful.

**The fix: query rewriting** (also called "contextual query reformulation").

```
   User turn 1: "What's the OAuth authorization code flow?"
       → Embed → Retrieve → Answer

   User turn 2: "What about the implicit flow?"
       → BEFORE embedding, ask the LLM to rewrite:
         "Rewrite this query as a standalone question
          given the chat history: <history>"
       → Rewritten: "What is the OAuth implicit flow?"
       → Embed the rewritten query → Retrieve → Answer
```

Without rewriting, multi-turn RAG breaks on the second message. Always rewrite.

### Code: full generation step

```python
from openai import OpenAI
from pydantic import BaseModel

client = OpenAI()

class RAGAnswer(BaseModel):
    answer: str
    citations: list[int]   # indices into the context blocks
    confidence: float       # 0.0 to 1.0

RAG_SYSTEM_PROMPT = """\
You answer questions using ONLY the provided context.

Rules:
1. If the answer is not in the context, set answer to:
   "I don't have information about that in my knowledge base."
2. Cite the context blocks you used by their numeric index.
3. Confidence: 0.9+ when the context directly answers; 0.5-0.8 when
   partial; below 0.5 when uncertain.
4. Be concise. No filler, no apologies.
"""

def generate_answer(question: str, chunks: list[dict]) -> RAGAnswer:
    context_block = "\n\n".join(
        f"[{i+1}] (source: {c['metadata']['source']})\n{c['text']}"
        for i, c in enumerate(chunks)
    )
    user_message = f"""\
<context>
{context_block}
</context>

Question: {question}
"""
    response = client.chat.completions.parse(
        model="gpt-5-mini",
        messages=[
            {"role": "system", "content": RAG_SYSTEM_PROMPT},
            {"role": "user",   "content": user_message},
        ],
        response_format=RAGAnswer,
        temperature=0,    # for consistency
    )
    return response.choices[0].message.parsed
```

Structured outputs + low temperature + explicit IDK rule = production-grade generation.

---

## 10. Evaluation: how to know your RAG actually works

You cannot improve RAG without measurement. This section is the difference between "I built a demo" and "I built a system."

### The two things to evaluate

```
   ┌────────────────────────────────────────┐
   │  1. RETRIEVAL QUALITY                  │
   │     "Did we get the right chunks?"     │
   │                                        │
   │     Metrics:                           │
   │       - Recall@k                       │
   │       - Precision@k                    │
   │       - Mean Reciprocal Rank (MRR)     │
   │       - NDCG@k                         │
   ├────────────────────────────────────────┤
   │  2. ANSWER QUALITY                     │
   │     "Did the LLM answer correctly?"    │
   │                                        │
   │     Metrics:                           │
   │       - Faithfulness (no hallucination)│
   │       - Relevance to the question      │
   │       - Citation correctness           │
   │       - Answer completeness            │
   └────────────────────────────────────────┘
```

**You must measure both separately.** If overall accuracy is low and you don't know which stage is failing, you'll spend weeks tuning the wrong thing.

### Building a golden set (start here)

Step zero of any RAG eval: build a golden set of 20–100 questions with known correct answers.

```python
GOLDEN_SET = [
    {
        "question": "What's the maximum session timeout for OAuth tokens?",
        "answer_must_contain": ["8 hours", "absolute", "idle"],
        "answer_must_not_contain": ["24 hours", "indefinite"],
        "expected_source_files": ["docs/auth/oauth.md", "docs/auth/sessions.md"],
    },
    {
        "question": "How do I rotate API keys?",
        "answer_must_contain": ["rotate", "endpoint", "new key"],
        "expected_source_files": ["docs/api/keys.md"],
    },
    # ... 20+ more
]
```

You can generate the golden set semi-automatically: ask an LLM to write Q&A pairs from your docs, then manually clean them. 1–2 hours of work, lasting value forever.

### Retrieval metric: Recall@k

The fraction of golden questions where at least one expected source is in the top-k retrieved chunks.

```python
def recall_at_k(golden_set, retriever, k=10):
    hits = 0
    for q in golden_set:
        retrieved_sources = {c["metadata"]["source"] for c in retriever(q["question"], k=k)}
        if any(src in retrieved_sources for src in q["expected_source_files"]):
            hits += 1
    return hits / len(golden_set)
```

This single number is the most important RAG metric in your project. Get it as high as possible (>90% for k=10 is a reasonable bar).

### Answer metric: LLM-as-judge

For answer quality, you can't do exact string matching. Use another LLM as a judge:

```python
JUDGE_PROMPT = """\
You are evaluating a RAG system's answer. Score each on 1-5:

1. FAITHFULNESS: is every claim supported by the context?
2. RELEVANCE: does the answer address the question?
3. COMPLETENESS: does the answer cover the key points expected?
4. CITATIONS: are the citations correct (and absent when answer is IDK)?

QUESTION: {question}
EXPECTED KEYWORDS: {must_contain}
EXPECTED SOURCES: {expected_sources}

CONTEXT GIVEN TO MODEL:
{context}

MODEL'S ANSWER:
{answer}

Return JSON: {{"faithfulness":N, "relevance":N, "completeness":N,
              "citations":N, "reasoning":"..."}}
"""
```

Use a stronger model than your generation model as judge (e.g., generation = GPT-5 mini, judge = GPT-5 or Claude Opus). The judge spends more tokens per answer than the generator, but you only run it during eval, not at user-query time.

### Tools to graduate to

- **Ragas** ([github.com/explodinggradients/ragas](https://github.com/explodinggradients/ragas)) — popular OSS framework with metrics like faithfulness, answer relevancy, context precision. Use it when your eval needs grow beyond a Python script.
- **LangSmith** (LangChain) — commercial; deep tracing of every retrieval and generation, with eval dashboards.
- **Braintrust** — commercial; good for prompt + RAG evals together.
- **Phoenix (Arize)** — open-source observability with built-in RAG eval.

For the first 10 weeks, a Python script over a JSON golden set is plenty. Reach for these tools when the script becomes a chore.

### The eval flywheel

```
   ┌────────────────────────────────────┐
   │ Ship change to RAG                 │
   └──────────────┬─────────────────────┘
                  ▼
   ┌────────────────────────────────────┐
   │ Run eval suite (Recall@10,         │
   │ answer-quality LLM-judge)          │
   └──────────────┬─────────────────────┘
                  ▼
   ┌────────────────────────────────────┐
   │ Inspect failures: which questions  │
   │ broke? Retrieval or generation?    │
   └──────────────┬─────────────────────┘
                  ▼
   ┌────────────────────────────────────┐
   │ Form hypothesis, change one thing  │
   └──────────────┬─────────────────────┘
                  ▼
   ┌────────────────────────────────────┐
   │ Re-run eval — did the number move? │
   └──────────────┬─────────────────────┘
                  ▼
              Loop. Forever.
```

This is the engineering loop. Without it, RAG tuning is astrology.

---

## 11. Advanced patterns (when basics aren't enough)

Save these for when your eval set says basic RAG isn't enough. Don't reach for them prematurely.

### Pattern 1: Query rewriting / expansion

If users ask short, ambiguous queries, expand or rewrite them before retrieval.

```
   User: "auth error"
   LLM rewrites: ["authentication error", "OAuth authorization failure",
                  "JWT validation error"]
   → Run vector search on all three → fuse with RRF
```

Cheap, often a 5–10% recall improvement.

### Pattern 2: HyDE (Hypothetical Document Embeddings)

Before searching, ask the LLM to generate a **hypothetical answer** to the user's question. Embed the hypothetical answer. Use *its* embedding for retrieval.

The intuition: answers are closer in embedding space to other answers than questions are to answers. So a fake answer is a better query than the literal question.

```
   User: "How do I reset my password?"
   LLM: "To reset your password, go to the login page and click..."
   → Embed THIS instead of the question
   → Retrieve
   → Answer with the real chunks
```

Useful when users ask short, abstract queries. Costs one extra LLM call. Often a 10%+ recall lift in casual-query domains.

### Pattern 3: Parent-child chunking (small-to-big)

```
   INDEXING:
     - Split each doc into SMALL chunks (e.g., 100 tokens).
     - Embed and index the SMALL chunks.
     - Also store a LARGER parent chunk (~1000 tokens) per small chunk,
       containing the surrounding context.

   QUERY-TIME:
     - Retrieve the SMALL chunks (precise).
     - But for each match, return its PARENT (richer context).
```

Best of both worlds: precise retrieval + enough context for the LLM to answer well. Standard pattern in LlamaIndex (`SentenceWindowRetriever`).

### Pattern 4: Multi-query retrieval

Generate N variants of the user's question (paraphrases), retrieve for each, fuse results.

```
   User: "How does our rate limiter work?"

   LLM generates variants:
     - "Explain the rate limiting architecture"
     - "What algorithm does the rate limiter use?"
     - "How are rate limits configured per endpoint?"

   → Embed each → Retrieve top-k for each → RRF fuse → top-N
```

Costs more, but covers more of the embedding space. Useful when users phrase the same intent very differently.

### Pattern 5: Routing / multi-index

If you have several knowledge bases (engineering wiki, HR docs, customer support), don't search them all every time. Route first.

```
   User question
        ↓
   ┌─────────────────────┐
   │ Router LLM:          │
   │ "Which knowledge     │
   │  base is relevant?"  │
   └──────────┬──────────┘
              │
      ┌───────┼───────┐
      ▼       ▼       ▼
   Eng wiki  HR docs  Support KB
   (its own  (its own (its own
    vector    vector   vector
    index)    index)   index)
```

Smaller indexes → more precise retrieval. Faster, cheaper, often better.

### Pattern 6: Self-RAG / corrective RAG

After retrieval, the LLM judges whether the retrieved chunks are sufficient. If not, it issues new search queries (sometimes against the web) and tries again. Effectively a small agent loop on top of RAG.

This is genuinely powerful but is also when RAG starts becoming an agent (Stage 5). Don't build this for v1 — get basic RAG with re-ranking working first.

### Pattern 7: Graph RAG

When your data has rich relationships (entities, references, code dependencies), embedding-based retrieval misses the graph structure. Graph RAG builds an entity/relationship graph from documents and retrieves *connected subgraphs* relevant to the query.

Powerful for: legal contracts (entities + relationships), academic papers (citations), code (call graphs). Overkill for plain docs.

### When NOT to add complexity

If your eval set is at 85% recall and you're tempted to add HyDE, multi-query, and Graph RAG — stop. **You probably have data quality or chunking issues that no architectural fix will solve.** Fix those first.

The order of operations when RAG is underperforming:

1. Look at the failing cases manually. Are they retrievable at all from your corpus?
2. Improve your data: better source docs, better metadata, better chunking.
3. Add hybrid search.
4. Add re-ranking.
5. *Then* consider advanced retrieval patterns.

90% of RAG projects can stop at step 4.

---

## 12. Failure modes: a field guide

Real RAG failures, with diagnoses. When your system is broken, this is your debugging playbook.

### Failure 1: "The answer is in the docs but RAG can't find it"

**Symptoms:** Eval shows low recall@k. Looking at the question and the expected source, the source is obviously the right doc.

**Diagnoses:**
- **Embedding mismatch**: query phrasing differs too much from source phrasing. Fix: HyDE, query expansion, hybrid search.
- **Chunking issue**: the answer is split across two chunks; neither alone is retrievable. Fix: larger chunks, more overlap, parent-child pattern.
- **Vocabulary gap**: the source uses jargon the user doesn't. Fix: index synonyms; include section headers in chunks.
- **Bad indexing**: did you actually index this doc? Check.

### Failure 2: "Right chunks, wrong answer"

**Symptoms:** Manual inspection shows the right chunks made it into context, but the LLM still hallucinates or gives a wrong answer.

**Diagnoses:**
- **Missing IDK instruction**: the model invented an answer. Fix: add explicit "say IDK" rule.
- **Lost in the middle**: chunk was buried in the middle of long context. Fix: re-rank to top-3-5 only.
- **Conflicting chunks**: two retrieved chunks contradict each other. Fix: add a "if sources conflict, note both" rule; consider freshness filters.
- **Weak model**: occasionally a small model just can't reason over the context. Try a stronger model on failing cases.

### Failure 3: "Off-topic chunks dominate"

**Symptoms:** Top-k retrieval is full of plausible-but-irrelevant chunks. Real answer is at rank 15.

**Diagnoses:**
- **First-stage retrieval is too coarse**. Fix: add re-ranking (most likely fix; biggest lift).
- **Metadata mismatch**: filters not applied, so unrelated docs leak in. Fix: enforce filters.

### Failure 4: "Citations are wrong / made up"

**Symptoms:** Model cites chunk [3] but the answer came from chunk [1], or cites a chunk number that doesn't exist.

**Diagnoses:**
- **Use structured outputs** for citations (`citations: list[int]` in a schema). Drastically reduces this.
- **Lower temperature** to 0.
- **Number chunks clearly** in the prompt; ambiguous numbering causes errors.

### Failure 5: "Answers are too verbose / too short"

**Symptoms:** User wants concise answers; RAG returns essays. Or vice versa.

**Diagnoses:**
- **Prompt is silent on length**. Fix: "Answer in 2–3 sentences." or "Provide a 1–2 paragraph explanation with examples."
- **Few-shot examples** showing desired length are the most reliable fix.

### Failure 6: "Multi-turn queries break the system"

**Symptoms:** First question works; "what about the second one?" returns garbage.

**Diagnosis:** No query rewriting.

**Fix:** Implement query rewriting (§9). Run the rewrite as a cheap LLM call before retrieval.

### Failure 7: "It worked yesterday, broke today"

**Symptoms:** No code change but quality regressed.

**Diagnoses:**
- **Index drift**: source docs changed but the index didn't update. Fix: re-index on a schedule.
- **Embedding model version**: provider rolled out a new version. Fix: pin version explicitly.
- **LLM behavior change**: model was updated. Fix: pin model version; build prompt regression tests.

### Failure 8: "It's too slow"

**Symptoms:** RAG query takes 5+ seconds, users complain.

**Diagnoses by component**:
- **Embedding the query**: 50–200ms typically. If slower, use a faster model or cache common queries.
- **Vector search**: 50–500ms. If slower, your HNSW index parameters need tuning, or you have too many vectors.
- **Re-ranking**: 100–500ms. The biggest variable cost. Reduce candidate count (re-rank 10, not 50).
- **LLM generation**: 1–5s. Stream the response so perceived latency is much lower.

For user-facing UIs: **always stream**. The first token in 800ms feels fast; waiting for the whole answer feels slow.

### Failure 9: "Costs are spiking"

**Symptoms:** Bill goes up 10x in a month.

**Diagnoses:**
- **Context bloat**: re-ranking but still passing all 20 chunks? Pass top 3–5.
- **No prompt caching**: same system prompt re-billed every call. Enable provider's prompt caching.
- **Multi-turn explosion**: chat history isn't trimmed; every turn is 50k tokens.
- **Embedding re-runs**: re-indexing the full corpus when only 5 docs changed. Implement incremental indexing.

---

## 13. Mini-project: "Chat with my docs" end-to-end

Time to build the thing. ~3–4 hours of focused work. By the end, you'll have a working RAG system that answers questions about a corpus you provide.

### What you'll build

A CLI tool: you point it at a folder of markdown/text files; it indexes them; you ask questions; it returns answers with citations.

### Architecture (recap)

```
   index.py   →   reads ./docs/**, chunks, embeds, stores in Postgres
   query.py   →   takes a question, retrieves, re-ranks, generates answer
   eval.py    →   runs your golden set, reports recall@k and answer quality
```

### Prerequisites

```bash
# 1. Postgres with pgvector (locally via Docker)
docker run -d --name pgvector \
    -e POSTGRES_PASSWORD=postgres \
    -p 5432:5432 \
    pgvector/pgvector:pg16

# 2. Install Python deps
pip install openai psycopg[binary] pgvector cohere pydantic python-dotenv

# 3. Set env vars
export OPENAI_API_KEY=sk-...
export COHERE_API_KEY=...     # optional, for re-ranking
export DATABASE_URL=postgresql://postgres:postgres@localhost/postgres
```

### Step 1: Database setup (`setup.sql`)

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE IF NOT EXISTS chunks (
    id            BIGSERIAL PRIMARY KEY,
    source        TEXT NOT NULL,
    chunk_index   INTEGER NOT NULL,
    chunk_text    TEXT NOT NULL,
    metadata      JSONB NOT NULL DEFAULT '{}'::jsonb,
    embedding     VECTOR(1536),
    created_at    TIMESTAMPTZ DEFAULT now(),
    UNIQUE (source, chunk_index)
);

CREATE INDEX IF NOT EXISTS chunks_embedding_idx
    ON chunks USING hnsw (embedding vector_cosine_ops);

CREATE INDEX IF NOT EXISTS chunks_metadata_gin_idx
    ON chunks USING gin (metadata);
```

Run: `psql $DATABASE_URL -f setup.sql`

### Step 2: `index.py`

```python
import os, glob, json
from pathlib import Path
from openai import OpenAI
import psycopg
from pgvector.psycopg import register_vector

CHUNK_SIZE = 500   # tokens (approx; 4 chars/token)
CHUNK_OVERLAP = 50

client = OpenAI()


def recursive_split(text, chunk_size=CHUNK_SIZE, overlap=CHUNK_OVERLAP):
    seps = ["\n\n", "\n", ". ", " "]
    cs = chunk_size * 4
    ov = overlap * 4

    def _split(t, ss):
        if len(t) <= cs:
            return [t]
        if not ss:
            return [t[i:i+cs] for i in range(0, len(t), cs - ov)]
        sep = ss[0]
        parts = t.split(sep)
        out, buf = [], ""
        for p in parts:
            cand = buf + (sep if buf else "") + p
            if len(cand) <= cs:
                buf = cand
            else:
                if buf:
                    out.append(buf)
                if len(p) > cs:
                    out.extend(_split(p, ss[1:]))
                    buf = ""
                else:
                    buf = p
        if buf:
            out.append(buf)
        return out

    return _split(text, seps)


def embed_batch(texts):
    resp = client.embeddings.create(model="text-embedding-3-small", input=texts)
    return [d.embedding for d in resp.data]


def index_folder(folder: str):
    conn = psycopg.connect(os.environ["DATABASE_URL"])
    register_vector(conn)

    files = glob.glob(f"{folder}/**/*.md", recursive=True) + \
            glob.glob(f"{folder}/**/*.txt", recursive=True)

    for filepath in files:
        text = Path(filepath).read_text()
        chunks = recursive_split(text)
        if not chunks:
            continue

        # Embed in batches of 100
        for batch_start in range(0, len(chunks), 100):
            batch = chunks[batch_start:batch_start+100]
            vectors = embed_batch(batch)

            with conn.cursor() as cur:
                for i, (chunk_text, vec) in enumerate(zip(batch, vectors)):
                    chunk_idx = batch_start + i
                    metadata = {
                        "source": filepath,
                        "title": Path(filepath).stem,
                    }
                    cur.execute(
                        """
                        INSERT INTO chunks (source, chunk_index, chunk_text, metadata, embedding)
                        VALUES (%s, %s, %s, %s, %s)
                        ON CONFLICT (source, chunk_index) DO UPDATE
                          SET chunk_text = EXCLUDED.chunk_text,
                              metadata = EXCLUDED.metadata,
                              embedding = EXCLUDED.embedding
                        """,
                        (filepath, chunk_idx, chunk_text, json.dumps(metadata), vec),
                    )
            conn.commit()
        print(f"Indexed {filepath}: {len(chunks)} chunks")


if __name__ == "__main__":
    import sys
    index_folder(sys.argv[1] if len(sys.argv) > 1 else "./docs")
```

Run: `python index.py ./docs`

### Step 3: `query.py`

```python
import os, json
from openai import OpenAI
from pydantic import BaseModel
import psycopg
from pgvector.psycopg import register_vector
import cohere

openai_client = OpenAI()
cohere_client = cohere.Client(api_key=os.environ.get("COHERE_API_KEY"))


class RAGAnswer(BaseModel):
    answer: str
    citations: list[int]
    confidence: float


def embed_query(question: str) -> list[float]:
    resp = openai_client.embeddings.create(
        model="text-embedding-3-small",
        input=question,
    )
    return resp.data[0].embedding


def retrieve(query_embedding, k=20):
    conn = psycopg.connect(os.environ["DATABASE_URL"])
    register_vector(conn)
    with conn.cursor() as cur:
        cur.execute(
            """
            SELECT chunk_text, metadata, 1 - (embedding <=> %s::vector) AS sim
            FROM chunks
            ORDER BY embedding <=> %s::vector
            LIMIT %s
            """,
            (query_embedding, query_embedding, k),
        )
        rows = cur.fetchall()
    return [{"text": r[0], "metadata": r[1], "similarity": r[2]} for r in rows]


def rerank(question: str, candidates, top_n=5):
    if not cohere_client.api_key:
        return candidates[:top_n]
    resp = cohere_client.rerank(
        model="rerank-english-v3.0",
        query=question,
        documents=[c["text"] for c in candidates],
        top_n=top_n,
    )
    return [candidates[r.index] for r in resp.results]


SYSTEM = """\
You answer questions using ONLY the provided <context> blocks.

Rules:
1. If the answer is not in the context, set answer to:
   "I don't have information about that in my knowledge base."
2. Cite the context block indices you used in the `citations` field.
3. Be concise. No filler.
"""


def generate(question, chunks):
    context = "\n\n".join(
        f"[{i+1}] (source: {c['metadata'].get('source')})\n{c['text']}"
        for i, c in enumerate(chunks)
    )
    resp = openai_client.chat.completions.parse(
        model="gpt-5-mini",
        messages=[
            {"role": "system", "content": SYSTEM},
            {"role": "user", "content": f"<context>\n{context}\n</context>\n\nQuestion: {question}"},
        ],
        response_format=RAGAnswer,
        temperature=0,
    )
    return resp.choices[0].message.parsed


def ask(question: str):
    q_emb = embed_query(question)
    candidates = retrieve(q_emb, k=20)
    top = rerank(question, candidates, top_n=5)
    answer = generate(question, top)

    print(f"\n📝 Answer: {answer.answer}")
    print(f"🎯 Confidence: {answer.confidence:.2f}")
    print("📚 Sources cited:")
    for i in answer.citations:
        if 1 <= i <= len(top):
            print(f"   [{i}] {top[i-1]['metadata'].get('source')}")
    return answer


if __name__ == "__main__":
    import sys
    if len(sys.argv) > 1:
        ask(" ".join(sys.argv[1:]))
    else:
        while True:
            q = input("\n❓ Question (Ctrl-C to exit): ")
            if not q:
                continue
            ask(q)
```

Run: `python query.py "How does authentication work?"`

### Step 4: `eval.py`

```python
import json
from query import embed_query, retrieve, rerank, generate

GOLDEN = json.load(open("golden_set.json"))


def recall_at_k(k=10):
    hits = 0
    for case in GOLDEN:
        q_emb = embed_query(case["question"])
        candidates = retrieve(q_emb, k=k)
        retrieved_sources = {c["metadata"].get("source") for c in candidates}
        if any(src in retrieved_sources for src in case["expected_sources"]):
            hits += 1
        else:
            print(f"❌ MISS: {case['question'][:60]}")
            print(f"   expected: {case['expected_sources']}")
            print(f"   retrieved: {list(retrieved_sources)[:5]}")
    rate = hits / len(GOLDEN)
    print(f"\nRecall@{k}: {rate:.0%} ({hits}/{len(GOLDEN)})")
    return rate


def answer_quality():
    """Crude: does the answer contain expected keywords?"""
    passed = 0
    for case in GOLDEN:
        q_emb = embed_query(case["question"])
        candidates = retrieve(q_emb, k=20)
        top = rerank(case["question"], candidates, top_n=5)
        answer = generate(case["question"], top)
        ok = all(kw.lower() in answer.answer.lower()
                 for kw in case.get("answer_must_contain", []))
        if ok:
            passed += 1
        else:
            print(f"❌ Answer miss: {case['question'][:60]}")
            print(f"   missing: {[kw for kw in case['answer_must_contain'] if kw.lower() not in answer.answer.lower()]}")
            print(f"   got: {answer.answer[:120]}")
    rate = passed / len(GOLDEN)
    print(f"\nAnswer keyword recall: {rate:.0%} ({passed}/{len(GOLDEN)})")


if __name__ == "__main__":
    recall_at_k(10)
    answer_quality()
```

And a starter `golden_set.json`:

```json
[
    {
        "question": "How do I configure OAuth?",
        "expected_sources": ["./docs/auth/oauth.md"],
        "answer_must_contain": ["client_id", "redirect_uri"]
    },
    {
        "question": "What is the rate limit per user?",
        "expected_sources": ["./docs/api/rate-limits.md"],
        "answer_must_contain": ["per minute", "exceeded"]
    }
]
```

Run: `python eval.py`

### What you've just built

A complete, production-shaped RAG system: chunking, batched embedding, pgvector storage, two-stage retrieval (vector + Cohere rerank), structured-output generation with citations and IDK handling, and an eval harness.

This is what AI engineers ship. Many "AI features" you've seen in real products are roughly this, with a nicer UI on top.

### Ways to extend it (optional, in priority order)

1. **Add hybrid search**: implement BM25 via Postgres FTS, fuse with RRF.
2. **Add query rewriting** for multi-turn chat.
3. **Add a Streamlit / FastAPI UI** so non-devs can use it.
4. **Add incremental indexing**: re-index only changed files (hash-based).
5. **Add observability**: log every query, retrieval results, answer, and latency.

---

## 14. Hands-on labs

### Lab 1: Chunking comparison (60 min)

Take one long document (a 30+ page PDF or markdown). Index it three ways:

1. Fixed-size 200 tokens, no overlap.
2. Recursive 500 tokens, 50 overlap.
3. Recursive 1500 tokens, 100 overlap.

Build a 5-question golden set against it. Run recall@5 for each. Observe which works best for your content.

**Deliverable:** a one-paragraph note on what you learned about chunk size for your data.

### Lab 2: Embedding model bake-off (45 min)

Same corpus, same golden set. Compare:

- `text-embedding-3-small`
- `text-embedding-3-large`
- `bge-m3` (run locally via sentence-transformers)

Which has the best recall@5? Is the difference meaningful (>5 pp)?

**Deliverable:** a small table with recall numbers; pick a winner with reasoning.

### Lab 3: Re-ranking lift (30 min)

Take your existing RAG. Run eval with re-ranking on / off. Report the lift.

**Expected observation:** 5–15 pp lift in recall (especially recall@5).

### Lab 4: Hybrid search (90 min)

Add Postgres full-text search alongside your vector search. Implement RRF. Eval again.

**Expected observation:** hybrid helps most on queries with proper nouns / exact codes; little change on chatty queries.

### Lab 5: Build the mini-project (3 hours)

If you skipped §13, do it now. Don't just copy-paste — type it. Read every line. Make it work end-to-end on your own corpus.

**Deliverable:** working RAG system + golden set + eval results. This is your portfolio piece.

### Lab 6: Failure investigation (60 min)

Pick 5 questions from your golden set where your RAG fails. For each, diagnose using §12. Note: was it retrieval or generation? What category of failure? Fix one and verify the eval moves.

**Deliverable:** a one-page failure analysis you could show to a senior engineer.

### Lab 7: Cost & latency profile (45 min)

For your RAG, measure for one query:

- Embedding latency
- Vector search latency
- Re-rank latency
- LLM latency
- Total token cost (input + output)

Now estimate: at 10k queries/day, what's the monthly cost?

**Deliverable:** a small profile table + cost estimate, with notes on which component is the biggest contributor.

---

## 15. Common anti-patterns

### Anti-pattern 1: No eval set

Building RAG without a golden set is building blind. You'll spend weeks tuning the wrong thing. **Fix:** day-zero golden set, even just 10 questions.

### Anti-pattern 2: Chunking too aggressively

500 tokens is the safe default for a reason. 100-token chunks lose context. 5000-token chunks defeat the precision of retrieval. **Fix:** start with the standard; tune based on eval, not intuition.

### Anti-pattern 3: No metadata

Chunks without metadata can't be filtered, can't carry citations, can't carry permissions. **Fix:** always attach `{source, title, section, date, doc_type}` minimum.

### Anti-pattern 4: Different embedding models for indexing vs querying

The most embarrassing RAG bug. Similarities become random. **Fix:** lock the embedding model in config; treat it like a database schema migration.

### Anti-pattern 5: Skipping the "say IDK" instruction

Without it, your RAG hallucinates confidently when knowledge is missing. **Fix:** always include the explicit IDK rule, with an exact phrasing.

### Anti-pattern 6: Dumping all 50 retrieved chunks into context

Bigger is not better. Lost in the middle, expensive, slow. **Fix:** retrieve 20, re-rank to 5.

### Anti-pattern 7: Not re-ranking

The single highest-leverage RAG upgrade. Skip it and you'll be 5–15% behind teams that don't. **Fix:** add Cohere Rerank or a local cross-encoder.

### Anti-pattern 8: Treating multi-turn chat like single-turn

"What about the second one?" — embedded as-is — returns garbage. **Fix:** query rewriting.

### Anti-pattern 9: No observability

When users report bad answers, you have no way to debug. **Fix:** log full query, retrieved chunks, final prompt, answer — for every production call. (See Stage 7.)

### Anti-pattern 10: Premature complexity

Reaching for Graph RAG, agentic RAG, multi-vector before basic RAG + re-ranking is tuned. **Fix:** ladder of upgrades (§11); stop at step 4 unless data demands more.

---

## 16. Interview questions companies actually ask

### Conceptual

1. **"Explain RAG in one minute."**
   *Good answer:* Two phases. **Indexing (offline)**: chunk documents, embed chunks, store vectors. **Query-time (online)**: embed the user's question, retrieve top-k similar chunks via vector search (optionally + BM25, fused with RRF), re-rank with a cross-encoder, build a prompt with the question + top chunks + "cite sources / say IDK" rules, generate an answer with an LLM. It's search + LLM, glued together.

2. **"Why not just use a long context window and skip retrieval?"**
   *Good answer:* Three reasons. (1) **Cost**: long contexts are billed per token; RAG scales the corpus independently of cost-per-query. (2) **Quality**: long contexts suffer from lost-in-the-middle; retrieval surfaces the right chunks to the top. (3) **Freshness**: re-indexing is cheap; re-training is not. RAG separates knowledge from the model.

3. **"What's the difference between bi-encoders and cross-encoders, and where does each fit?"**
   *Good answer:* Bi-encoders embed query and chunk independently → pre-indexable, fast, used for first-stage retrieval. Cross-encoders take (query, chunk) together → much more accurate but cannot pre-index → used for re-ranking on a small candidate set.

4. **"Why is chunking the most underrated lever?"**
   *Good answer:* Because everything downstream is gated by retrieval quality, and retrieval quality is gated by whether the right information lives inside one chunk. Bad chunking can defeat the best model. Good chunking gives small models superpowers.

5. **"When would you reach for a graph database vs. a vector database for RAG?"**
   *Good answer:* Vector DB is the default; works for any text corpus. Reach for graph (Graph RAG) when the data is heavily relational and queries need to traverse relationships — legal contracts (entities + clauses), academic citations, code call graphs. Don't reach for it for plain docs.

### Practical

6. **"You're seeing 60% recall@10 on your eval set. Walk me through how you'd improve it."**
   *Good answer:* (1) Look at failures manually — are they retrievable at all? (2) Verify metadata + source coverage. (3) Try larger chunks + more overlap. (4) Add hybrid search (BM25 + vector + RRF). (5) Add a re-ranker. (6) Try a stronger embedding model. (7) If still failing on specific patterns, try query rewriting or HyDE. Always run the full eval between changes — don't stack untested changes.

7. **"Design RAG over 50k Confluence pages with team-level access control."**
   *Good answer:* Indexing: pull pages on a schedule (daily or webhook-driven); chunk recursively at 500/50; embed with `text-embedding-3-small`; store in pgvector with metadata including team(s), URL, last-updated. Query-time: filter by `metadata->>'team' IN (user_teams)` BEFORE vector search, retrieve top-20, re-rank to top-5, generate with cite-and-IDK prompt. Eval: 50-question golden set per major topic; recall@10 and answer-quality LLM-judge. Observability: log full query/retrieval/answer; track cost and latency.

8. **"How do you handle multi-turn chat in RAG?"**
   *Good answer:* Implement **query rewriting**: before embedding, ask the LLM to rewrite the latest user message as a standalone question, given the chat history. Embed the rewritten query. Otherwise "what about the second one?" returns garbage.

9. **"Your boss says 'just dump more chunks into the context'. Why is that wrong?"**
   *Good answer:* (a) Cost scales linearly with input tokens. (b) Lost-in-the-middle hurts accuracy. (c) Confuses the model with irrelevant chunks. The right move is fewer, more relevant chunks — which is what re-ranking gives you. Counterintuitive but well-established: less context, better answers, lower cost.

10. **"What metrics do you track in production for a RAG system?"**
    *Good answer:* Retrieval: recall@k against eval set (offline); query latency p50/p95; embedding API cost. Generation: answer-quality score (LLM-judge sampled); citation correctness; IDK rate (% answers that say "I don't know"); user feedback (thumbs up/down or implicit signals). Cost: tokens per query (input + output); monthly bill; cost per question. Failure: error rate; specific failure categories from logs.

### Senior-level / system design

11. **"Walk me through a RAG over a multi-tenant SaaS where each tenant has private docs."**
    *Hint:* Per-tenant namespacing (separate index/collection or a tenant_id column with strict filtering); per-tenant embeddings often the same model but separate retrieval; permissions enforced at retrieval (never trust the LLM); separate eval sets per tenant if their domains differ; rate limiting per tenant; cost attribution.

12. **"How would you handle RAG over docs that change daily?"**
    *Hint:* Webhook-driven or incremental indexing (hash each chunk; re-embed only on change); soft-delete old vectors; freshness boost in retrieval (recent docs rank higher via metadata-aware scoring); re-rank with date as a feature; periodic full re-index as backup. Eval set updated weekly to reflect new content.

13. **"What's the cost-quality tradeoff in your RAG decisions?"**
    *Hint:* Bigger embeddings (3-large vs 3-small) → 6x cost, 1–3 pp recall lift; managed re-ranking (Cohere) → adds $/query but 5–15 pp lift; frontier LLM for generation → 10x cost, modest quality lift if retrieval is good. Frame answers in terms of "$ per question" and "user-visible quality lift" — that's the senior framing.

14. **"You need to ship a RAG MVP in 2 weeks. Hardest call you'd make?"**
    *Hint:* Cut scope: one corpus, no multi-turn, no fine-grained permissions in v1. Use managed services (OpenAI embeddings + pgvector + Cohere rerank) — no self-hosting. Skip advanced patterns. Build a 30-question golden set day one. Production fundamentals (eval + logging) over feature breadth.

15. **"When does it stop being RAG and start being an agent?"**
    *Hint:* When retrieval is no longer a single fixed step but the model is deciding *what to search for* and *whether to search again*. Self-RAG, corrective RAG, tool-using agents over a KB — all are points on this spectrum. The dividing line is whether the model controls the retrieval loop or you do.

---

## 17. Self-check: are you done with Stage 4?

You're ready for Stage 5 (agents) when you can:

- [ ] Draw the full RAG pipeline from memory in under a minute (indexing + query-time).
- [ ] Explain why RAG = search + LLM, not magic.
- [ ] List 5 chunking strategies and pick the right one for a given data type.
- [ ] Set up pgvector and write the schema + a query, without copying.
- [ ] Implement RRF for hybrid search.
- [ ] Explain bi-encoder vs cross-encoder and where each fits.
- [ ] Run an end-to-end RAG eval and report recall@k.
- [ ] Diagnose a failing RAG: retrieval issue vs generation issue.
- [ ] Implement structured-output generation with citations and IDK handling.
- [ ] Have built the §13 mini-project on a real corpus (yours or sample) and run the eval.

8+ checked → you're ready for Stage 5 (agents). The mini-project (item 10) is the most important; if you skipped it, do it before moving on.

Don't move on if you've only "read" this material. RAG without hands-on is theory. **Build the project.**

---

## Further reading

- **[Anthropic — Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval)** — a strong recent technique for adding context to chunks. Worth implementing once your basics work.
- **[Jerry Liu (LlamaIndex) on RAG](https://www.llamaindex.ai/blog)** — practitioner blog with patterns.
- **[Pinecone Learn](https://www.pinecone.io/learn/)** — free, well-written introductions to most RAG topics.
- **[The Engineer's Guide to RAG (Jason Liu)](https://jxnl.co/)** — opinionated, practical.
- **[Ragas docs](https://docs.ragas.io/)** — when you're ready to graduate from a Python eval script.
- **[Cohere — Reranker docs](https://docs.cohere.com/docs/rerank-2)** — for understanding the re-ranking step deeper.

---

## What's next

Stage 4 makes AI answer questions. Stage 5 makes AI **take actions**. You'll build agents that don't just look things up but actually do things — call APIs, write to databases, run code, send messages. RAG often lives *inside* an agent as one of its tools.

See [AI_ROADMAP.md](./AI_ROADMAP.md) for the full path, or jump to [AI_STAGE5_AGENTS.md](./AI_STAGE5_AGENTS.md) (coming next).
