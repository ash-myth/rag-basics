# 🤖 LangGraph RAG Demo

A minimal agentic AI system that combines a **FAISS vector knowledge base** with **LangGraph** to answer AI/ML questions via RAG and handle arithmetic — all through a clean Streamlit UI.

Built with LangGraph + Groq + Google Gemini Embeddings + FAISS.

---

## Features

- **RAG pipeline** — searches a local FAISS index built from your own `.txt` docs
- **Calculator tools** — add, multiply, divide via structured tool calling
- **LangGraph agent** — ReAct-style loop that decides when to retrieve vs. compute vs. answer directly
- **Tool Trace sidebar** — live view of every tool call, input, and output per turn
- **Streamlit UI** — clean chat interface with persistent session history

---

## Project Structure

```
agentic-ai-basic/
├── agent.py          # LangGraph graph, tools (search_docs, math), and agent compilation
├── app.py            # Streamlit UI — chat interface + tool trace sidebar
├── ingest.py         # Document loader, chunker, embedder, FAISS index builder
├── sample_docs/      # Drop your .txt knowledge base files here
├── faiss_index/      # Auto-generated after running ingest.py (gitignored)
├── .env              # API keys (gitignored)
└── requirements.txt  # Dependencies
```

---

## Setup

### 1. Clone and install

```bash
git clone https://github.com/ash-myth/rag-basics.git
cd rag-basics
pip install -r requirements.txt
```

### 2. Set up API keys

Create a `.env` file in the project root:

```env
GOOGLE_API_KEY=your_gemini_api_key
GROQ_API_KEY=your_groq_api_key
```

- **Google API key** — used for Gemini embeddings (`gemini-embedding-001`). Get it at [aistudio.google.com](https://aistudio.google.com)
- **Groq API key** — used for the LLM backbone. Get it at [console.groq.com](https://console.groq.com)

### 3. Add your documents

Place `.txt` files inside `sample_docs/`. The ingestion pipeline will chunk and embed all of them.

> **Free tier note:** The Gemini Embeddings free tier allows 100 requests/minute. If your document splits into more than ~90 chunks, the ingest will hit a rate limit. Either trim your docs or add batching with a delay.

### 4. Build the FAISS index

```bash
python ingest.py
```

This loads all `.txt` files from `sample_docs/`, splits them into chunks (~500 chars), embeds them with Gemini, and saves the FAISS index to `faiss_index/`.

### 5. Run the app

```bash
streamlit run app.py
```

---

## How It Works

```
User Query
    │
    ▼
LangGraph Agent (llm_call node)
    │
    ├─── Math question? ──► add / multiply / divide tool
    │
    └─── AI/ML question? ──► search_docs tool
                                    │
                                    ▼
                            FAISS vector search
                            (top-3 chunks retrieved)
                                    │
                                    ▼
                            Chunks injected into prompt
                                    │
                                    ▼
                            LLM generates grounded answer
```

The agent runs a ReAct loop — it can call multiple tools in sequence before returning a final answer. Every tool call and its result is captured and displayed in the sidebar.

---

## Chunking Strategy

Documents are split on double newlines (`\n\n`) into paragraph-level chunks, with a 50-character overlap carried over between chunks to preserve context across boundaries. Chunks are capped at 500 characters.

---

## Stack

| Component | Technology |
|---|---|
| Agent framework | LangGraph |
| LLM | Groq (`openai/gpt-oss-20b`) |
| Embeddings | Google Gemini (`gemini-embedding-001`) |
| Vector store | FAISS (local, no server needed) |
| UI | Streamlit |
| Environment | python-dotenv |

---

## Example Queries

- *"What is the difference between RAG and fine-tuning?"*
- *"How does Flash Attention work?"*
- *"What is 347 multiplied by 19?"*
- *"Explain the KV cache and why it matters for inference."*

---

## Gitignore

Make sure your `.gitignore` includes:

```
.env
faiss_index/
__pycache__/
*.pyc
```
