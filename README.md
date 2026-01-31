# Codebase Q&A RAG Bot

This repository contains a code-focused Retrieval-Augmented Generation (RAG) system: a Codebase Q&A bot that indexes a software repository, retrieves precise code chunks, and synthesizes answers that cite exact files and line ranges.

This README documents the Codebase Q&A workflow, how to run ingestion, verify metadata, run a local QA query, and suggestions for testing and deployment.

## Goals

- Enable developers to ask questions about a codebase and get precise, verifiable answers that reference specific files and line ranges.
- Avoid hallucinations by restricting answers to retrieved code chunks and requiring inline citations.
- Provide a reproducible local workflow built on top of the existing LangChain/Weaviate tooling.

## What this repo provides (Codebase Q&A features)

- `backend/ingest_codebase.py` — ingest a local repository into a Weaviate index using OpenAI embeddings and LangChain's `SQLRecordManager`.
- `backend/code_ingest_utils.py` — read files while preserving formatting, detect language, split into logical chunks (functions/classes/top-level sections), and emit `Document` objects with rich metadata.
- `backend/verify_weaviate_metadata.py` — query the Weaviate index and print sample objects to confirm metadata was saved.
- `backend/code_retriever.py` — retriever factory that performs simple intent detection and returns a Weaviate-backed retriever tuned for precision.
- `backend/code_qa.py` — CLI script that retrieves code chunks for a question and synthesizes answers constrained to retrieved context with inline citations.
- `.env.example` — example environment variables for local development.

## Architecture (focused)

- Ingestion: Read files from a codebase, chunk by logical boundaries, compute embeddings, persist vectors and structured metadata in Weaviate, and register records via the SQLRecordManager.
- Retrieval: Intent-aware retriever that adjusts search parameters (k) based on the question type (file location vs explanation).
- Synthesis: LLM prompt enforces usage of retrieved context only and requires inline citations in the form `[filename:start-end]`.

## Quickstart (local)

Prerequisites

- Python 3.11+
- Node.js + Yarn (if you want to run the frontend UI)
- A Weaviate instance (Weaviate Cloud or self-hosted)
- A Postgres-compatible database (Supabase recommended) for the record manager
- OpenAI API key (or another embedding/LLM provider — scripts use `langchain_openai` by default)

Setup

1. Clone the repo and create a Python venv:

```bash
git clone <this-repo>
cd chat-langchain
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -e .
```

2. Copy and edit `.env.example` and export your values, or export manually:

```bash
cp .env.example .env
export WEAVIATE_URL="https://<your-weaviate-cluster>"
export WEAVIATE_API_KEY="<your-weaviate-api-key>"
export RECORD_MANAGER_DB_URL="postgresql://user:pass@host:5432/db"
export OPENAI_API_KEY="sk-..."
export CODEBASE_PATH="/path/to/your/repo"  # optional; defaults to repo root
export WEAVIATE_INDEX_NAME="langchain_code"
```

3. Ingest your codebase into Weaviate:

```bash
python -m backend.ingest_codebase
```

What ingestion does

- Scans files in `CODEBASE_PATH` (only allowed extensions; excludes `node_modules`, `venv`, `.git`, `dist`, `build`, `__pycache__`).
- Reads each file preserving indentation and newlines.
- Splits files into logical chunks (functions/classes/top-level) using heuristics in `backend/code_ingest_utils.py`.
- Each chunk is stored as a vector with metadata: `path`, `filename`, `language`, `chunk_type`, `line_start`, `line_end`.

Verify metadata

```bash
export WEAVIATE_INDEX_NAME="langchain_code"
python -m backend.verify_weaviate_metadata
```

This prints a few sample objects and their metadata so you can confirm fields exist and are queryable.

Run a local QA query

```bash
python -m backend.code_qa "Where is ingestion implemented?"
```

- The script will detect the intent (`file`, `how`, `why`, `explain`), retrieve precise chunks using `backend/code_retriever.py`, construct a context with file headers, and call the LLM with a strict prompt requiring evidence-only answers and inline citations.

Design and implementation notes

- Chunking by logical boundaries avoids splitting a function in the middle; however the heuristics are conservative and may need tuning for some languages.
- Metadata is stored as attributes in Weaviate so retrieval can filter and the UI can display file paths and line ranges.
- Retrieval tuning is implemented in `backend/code_retriever.py` via `get_search_kwargs_for_intent` — adjust `k` and other parameters there.
- The QA synth prompt is intentionally strict to minimize hallucination; modify `backend/code_qa.py` `PROMPT` if you need different formatting or more verbose answers.

Frontend (optional, minimal)

- The repo's frontend app is a Next.js app under `frontend/`. To integrate the Codebase Q&A, surface inline citations returned by `code_qa` style responses and make source snippets expandable.
- Minimal change: when rendering an answer, show a list of cited files with clickable links that open the repository at the specified lines (if hosted on GitHub/Gitlab).

Testing & validation suggestions

- Phase 7: Build a small validation suite with a curated set of questions and expected file citations. Run ingestion on a small sample repo and assert that `code_qa` returns citations that reference the expected files.
- Add unit tests for `backend/code_ingest_utils.py` to validate chunk boundaries for sample code snippets.
- Optionally add an integration test that uses a local FAISS index (in-memory) so CI tests don't require Weaviate.

Security & privacy

- The ingestion stores code snippets and metadata in the vector store. Be careful with private keys and sensitive repositories. Use encrypted databases and restrict access to the Weaviate cluster.
- When using OpenAI or other hosted LLMs, be mindful of data sent to the provider.

Next steps (recommended)

1. Phase 6: Frontend integration — show file citations and expose expandable context snippets.
2. Phase 7: Add automated validation and smoke tests that assert citation correctness.
3. Optional: FAISS fallback for fully-local CI runs and developer experiments.

Contact / contributions

If you want help tuning chunking heuristics, retrieval parameters, or the synthesis prompt, open an issue or submit a PR with proposed changes and tests.

License

This project follows the license in the repo root.
# 🦜️🔗 Chat LangChain

This repo is an implementation of a chatbot specifically focused on question answering over the [LangChain documentation](https://python.langchain.com/).
Built with [LangChain](https://github.com/langchain-ai/langchain/), [LangGraph](https://github.com/langchain-ai/langgraph/), and [Next.js](https://nextjs.org).

Deployed version: [chat.langchain.com](https://chat.langchain.com)

> Looking for the JS version? Click [here](https://github.com/langchain-ai/chat-langchainjs).

The app leverages LangChain and LangGraph's streaming support and async API to update the page in real time for multiple users.

## Running locally

This project is now deployed using [LangGraph Cloud](https://langchain-ai.github.io/langgraph/cloud/), which means you won't be able to run it locally (or without a LangGraph Cloud account). If you want to run it WITHOUT LangGraph Cloud, please use the code and documentation from this [branch](https://github.com/langchain-ai/chat-langchain/tree/langserve).

> [!NOTE]
> This [branch](https://github.com/langchain-ai/chat-langchain/tree/langserve) **does not** have the same set of features.

Codebase Q&A (local ingestion)
--------------------------------

This repository now includes tooling to index a local codebase for Codebase Q&A (RAG adapted for source code). The ingestion preserves file structure and formatting, splits by logical code boundaries (functions/classes), and stores rich metadata so answers can cite exact files and line ranges.

Files added
- `.env.example` — example environment variables for local development (Weaviate, Supabase/RecordManager, OpenAI key, etc.).
- `backend/ingest_codebase.py` — script to ingest a local repository (CODEBASE_PATH) into Weaviate using the repo's existing embedding and record manager patterns.
- `backend/code_ingest_utils.py` — utilities that read files preserving indentation, detect language, split files into logical chunks (functions/classes) and emit LangChain `Document` objects with metadata (path, filename, language, chunk_type, line_start, line_end).
- `backend/verify_weaviate_metadata.py` — quick verification utility that queries the Weaviate index to ensure metadata fields were saved and prints sample objects.

Quick usage (local ingestion)
1. Install Python dependencies and the package in editable mode:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -e .
```

2. Copy and fill `.env.example` or export the required environment variables:

Required environment variables for ingestion:
- WEAVIATE_URL — Weaviate cluster URL
- WEAVIATE_API_KEY — Weaviate API key
- RECORD_MANAGER_DB_URL — Postgres/Supabase connection string (for LangChain SQLRecordManager)
- OPENAI_API_KEY — embeddings / LLM calls
- CODEBASE_PATH — path to the repository to index (optional; defaults to repo root)
- WEAVIATE_INDEX_NAME — optional index name

3. Run the ingestion script to index your repository:

```bash
export WEAVIATE_URL="https://<your-weaviate-cluster>"
export WEAVIATE_API_KEY="<your-key>"
export RECORD_MANAGER_DB_URL="postgresql://user:pass@host:port/db"
export OPENAI_API_KEY="sk-..."
export CODEBASE_PATH="/path/to/your/repo"
python -m backend.ingest_codebase
```

Notes on ingestion behavior
- The ingestion script preserves file formatting and stores each file as structured chunks. It detects language from file extensions and splits by logical boundaries (functions, classes, or top-level sections), then further splits large blocks into manageable line ranges.
- Each stored chunk includes metadata fields: `path`, `filename`, `language`, `chunk_type`, `line_start`, `line_end`, and the text content. These fields are stored as Weaviate attributes and are queryable.

Verify metadata in Weaviate
--------------------------------
After ingestion, you can verify that metadata fields were persisted with the included verification script:

```bash
export WEAVIATE_INDEX_NAME="langchain"  # or your index name
python -m backend.verify_weaviate_metadata
```

The verifier prints sample objects returned from Weaviate; each object should show the fields listed above and a `text` field with the chunk content.

Why this helps
- Chunking by logical code boundaries (not token counts) reduces the risk of cutting code mid-function.
- Rich metadata allows the retriever to filter and the synthesizer to cite exact files and line ranges in responses.

Next steps (recommended)
- Retrieval tuning: reduce `k` and prefer precision for code queries; implement intent-aware retriever strategies (file-level vs explanation queries).
- Synthesis constraints: create prompts that require answers to reference file names and line ranges and to avoid unsupported speculation.
- UI: expose file path and a link to the source in response cards so users can verify answers quickly.

Phase 4 & 5 — Retrieval tuning and answer synthesis (what we added)
-----------------------------------------------------------------
We've implemented Phase 4 (retrieval tuning) and Phase 5 (answer synthesis constraints) with two new modules:

- `backend/code_retriever.py` — a retriever factory tuned for code queries. It performs simple intent detection (file / how / why / explain) and returns a Weaviate-backed retriever with search kwargs optimized for precision (smaller `k` for file-location queries, slightly larger for explanatory queries).

- `backend/code_qa.py` — a small CLI QA runner that:
	- detects intent from the user question,
	- retrieves precise code chunks from Weaviate,
	- synthesizes an answer using an LLM with a strict prompt that requires the model to only use the retrieved context and to cite filenames and line ranges.

Quick usage (retrieval & QA)
1. Ensure environment variables are set (Weaviate + OpenAI):

```bash
export WEAVIATE_URL="https://<your-weaviate-cluster>"
export WEAVIATE_API_KEY="<your-key>"
export WEAVIATE_INDEX_NAME="langchain"
export OPENAI_API_KEY="sk-..."
```

2. Run a sample query using the CLI QA script (one-liner question):

```bash
python -m backend.code_qa "Where is ingestion implemented?"
```

What the QA script does
- Uses `code_retriever.detect_intent()` to classify the question into intents like `file`, `how`, `why`, or `explain`.
- Builds search kwargs tuned for that intent (for example, `k=3` for `file` queries to favor precision).
- Retrieves the top chunks and constructs a context with explicit headers `### filename [start-end]` followed by the chunk content.
- Sends the context + question to the LLM with a prompt that instructs the model to only answer from the context and to cite the source(s) inline as `[filename:start-end]`.

Why this reduces hallucinations
- By reducing `k` and enforcing a prompt that forbids adding external facts, we narrow the model's knowledge to retrieved evidence and require citations.
- Intent-aware retrieval adapts precision vs. coverage: `file` queries return fewer, highly-relevant chunks; `how`/`why` queries allow more chunks so explanatory connections can be made.

Customization points
- Tuning `k` and other Weaviate search parameters in `backend/code_retriever.py` (function `get_search_kwargs_for_intent`) directly affects precision vs recall.
- Prompt template in `backend/code_qa.py` (variable `PROMPT`) is intentionally strict; you can tweak wording to suit your team's style or to require different citation formats.

Next recommended steps after Phase 4 & 5
- Frontend: show inline citations and make source code expandable (Phase 6).
- Validation: build a question set and automated tests that assert the model cites the correct files (Phase 7).


If you need a fully self-hosted experience without Weaviate, consider adding a FAISS/Chroma fallback index and a small FastAPI wrapper to serve queries locally (the `langserve` branch is another option with fewer features).

## 📚 Technical description

There are two components: ingestion and question-answering.

Ingestion has the following steps:

1. Pull html from documentation site as well as the Github Codebase
2. Load html with LangChain's [RecursiveURLLoader](https://python.langchain.com/docs/integrations/document_loaders/recursive_url) and [SitemapLoader](https://python.langchain.com/docs/integrations/document_loaders/sitemap)
3. Split documents with LangChain's [RecursiveCharacterTextSplitter](https://python.langchain.com/api_reference/text_splitters/character/langchain_text_splitters.character.RecursiveCharacterTextSplitter.html)
4. Create a vectorstore of embeddings, using LangChain's [Weaviate vectorstore wrapper](https://python.langchain.com/docs/integrations/vectorstores/weaviate) (with OpenAI's embeddings).

Question-Answering has the following steps:

1. Given the chat history and new user input, determine what a standalone question would be using an LLM.
2. Given that standalone question, look up relevant documents from the vectorstore.
3. Pass the standalone question and relevant documents to the model to generate and stream the final answer.
4. Generate a trace URL for the current chat session, as well as the endpoint to collect feedback.

## Documentation

Looking to use or modify this Use Case Accelerant for your own needs? We've added a few docs to aid with this:

- **[Concepts](./CONCEPTS.md)**: A conceptual overview of the different components of Chat LangChain. Goes over features like ingestion, vector stores, query analysis, etc.
- **[Modify](./MODIFY.md)**: A guide on how to modify Chat LangChain for your own needs. Covers the frontend, backend and everything in between.
- **[LangSmith](./LANGSMITH.md)**: A guide on adding robustness to your application using LangSmith. Covers observability, evaluations, and feedback.
- **[Production](./PRODUCTION.md)**: Documentation on preparing your application for production usage. Explains different security considerations, and more.
- **[Deployment](./DEPLOYMENT.md)**: How to deploy your application to production. Covers setting up production databases, deploying the frontend, and more.
