# RAG ingestion and retrieval: 13-minute session runbook

**Content source:** `masterclass_content/week-02-rag-data-fluency/day1_masterclass2_content.md` (the file is named day1_masterclass2, not Day 2; it is the "RAG Architecture, Components & Vector Databases" session).
**Demo code:** `~/vscode/niit/coforge-fde/gitlab/northwind-console-week-3/`

## Timing

| Time | Block | What you do |
|---|---|---|
| 0:00–1:30 | Why RAG | Talk (Part 1) |
| 1:30–4:00 | Architecture | Talk (Parts 2 and 4), no code yet |
| 4:00–11:30 | Live demo | Steps 1–6 below |
| 11:30–13:00 | Close | Check-for-understanding and takeaway |

## Talk track (from the masterclass only)

**0:00 Why RAG (Part 1).** Say: "RAG is the difference between an open-book and a closed-book exam: the model answers from a book it can browse, not from memory." RAG connects a model to an external knowledge base without retraining it. It lowers hallucination risk but cannot make a model error-proof. `sources` is what turns "the model said so" into "the model said so, and here's the article it cites."

**1:30 Architecture (Part 2).** Draw or say the five stages: prompt → retriever queries the knowledge base → results go to the integration layer → augmented prompt → LLM generates. Retrieve, augment, generate. Then map the four components to files, so the demo lands:

| Component | File in the repo |
|---|---|
| Knowledge base | `resources/product_kb.json`, `past_tickets.json` |
| Retriever | `backend/rag/retrieval.py` |
| Integration layer | `draft_reply()` in `backend/main.py` (`rag/reply_service.py`) |
| Generator | `backend/providers/router.py` |

**3:00 Two ideas to plant before the demo.**
- Chunk size is a hyperparameter: too large and a chunk is too general to match a query, too small and it loses meaning. This repo uses 300 characters with 50 overlap.
- pgvector is an extension inside the Postgres you already run, "one engine, not two." It is the type of vector database that lives inside your relational DB.

## Pre-flight (do this BEFORE the session, not on stage)

```bash
cd ~/vscode/niit/coforge-fde/gitlab/northwind-console-week-3
source .venv/bin/activate
[ -f .env ] || cp .env.example .env
docker compose up -d
docker compose ps                  # postgres must show "healthy"
cd backend
python -m rag.embeddings           # first run downloads all-MiniLM-L6-v2; do it now so it's cached
```

Expected last line: `Embedded 2 texts, dimension=384`. Set your terminal font large. Keep two terminal tabs: one for `docker compose` (repo root), one for Python (`backend/`).

## Live demo (all Python commands run from `backend/` with the venv active)

**Step 1 · 4:00 · The knowledge base (ingestion input).**
```bash
python -m rag.ingestion
```
Shows `Loaded 22 documents from resources/` and the first three. Say: "12 KB articles and 10 past tickets, normalised into one Document shape. `id` and `title` are kept so a citation can point back to a real source."

**Step 2 · 5:00 · Chunking (before storing anything).**
```bash
python -m rag.chunking
```
Shows `22 documents -> N chunks (size=300, overlap=50)`. Say: "Chunking comes before embedding because the model's input is bounded. Same body text, smaller overlapping pieces." Do not skip this step: it is the chunk-size talking point made visible.

**Step 3 · 6:00 · Show the vector database is plain Postgres.**
```bash
cd .. && docker compose exec postgres psql -U northwind -d northwind -c "\dx"
cd backend
```
Point at the `vector` extension. Say: "The extension is named `vector`. IBM's own doc writes `pg_vector`, so check with `\dx` instead of taking either on faith."

**Step 4 · 7:00 · Embed and store (ingestion).**
First clear any earlier run, because `vector_store` appends and will duplicate rows if you run it twice:
```bash
cd .. && docker compose exec postgres psql -U northwind -d northwind -c "TRUNCATE document_chunks;" ; cd backend
python -m rag.vector_store
```
Shows `Stored N chunks in document_chunks`. Say: "Each chunk became a 384-dimension vector, the coordinates of its meaning, and was inserted next to its text, doc_id and title."
Then prove it:
```bash
cd .. && docker compose exec postgres psql -U northwind -d northwind -c "SELECT doc_id, position, left(text,60) AS text, vector_dims(embedding) AS dims FROM document_chunks LIMIT 5;" ; cd backend
```

**Step 5 · 8:30 · Retrieval (the core demo).**
```bash
python -m rag.retrieval
```
Default query is "How do I reset my password?"; expect `kb-0001` first, with the smallest distance. Say: "Lower distance means closer meaning. The query was embedded with the same model as the corpus, then pgvector returned the nearest three."

Then show retrieval is about meaning, not words. Use one-liners with different phrasing:
```bash
python -c "from rag.retrieval import search; [print(r.doc_id, round(r.distance,3), r.doc_title) for r in search('I forgot my login credentials', 3)]"
python -c "from rag.retrieval import search; [print(r.doc_id, round(r.distance,3), r.doc_title) for r in search('How do I rotate my API keys?', 3)]"
```
Expect `kb-0004` for the API-keys query. The "forgot my login credentials" query has no word in common with the password article; if `kb-0001` still ranks first, that is your "meaning, not keywords" moment. If it doesn't, say so and move on.

**Step 6 · 10:00 · What retrieval can't answer (honesty slide).**
```bash
python -c "from rag.retrieval import search; [print(r.doc_id, round(r.distance,3), r.doc_title) for r in search('What is your policy on accepting cryptocurrency payments?', 3)]"
```
No cryptocurrency document exists, yet search still returns three rows, with larger distances. Say: "A retriever always returns k results. Without a confidence threshold or a check on distance, it cannot say 'nothing relevant', which is why the grounded reply needs a refusal path."

**Optional if time remains (10:45):** the integration layer end to end, `python -m rag.eval` (recall@k against `resources/golden_set.json`, 15 queries). Skip it if running late.

## Close (11:30–13:00): check-for-understanding (from the masterclass)

1. Hybrid dense+sparse retrieval plus reranking: naive, advanced or modular? **Advanced.**
2. Which component turns text into vectors but stores nothing and runs no search? **The embedding model.**
3. True or false: pgvector is a separate database you install beside Postgres. **False, it is an extension inside it.**

**Takeaway:** "Retrieval is the open-book part. Chunking, embedding and the distance ordering decide whether the right page is open."

## Things that can go wrong

- **`document_chunks` doubles in size:** you ran `vector_store` twice. `TRUNCATE` and rerun.
- **Connection refused:** Docker isn't up or the container isn't healthy yet. Wait for `docker compose ps` to show healthy.
- **First run hangs:** the embedding model is downloading. Pre-flight avoids this; don't leave it for the session.
- **`ModuleNotFoundError: models` or `rag`:** you are not in `backend/`.
- **Retrieval returns the worst match first:** the sort order is wrong. In the repo it is `.asc()` on cosine distance, which is correct; smaller distance is closer.
- **If anything fails live:** say so, and switch to a recorded run or screenshots. Do not debug on stage.
