# 🐝 LibBee — KU Library AI Assistant

> **An AI-powered library assistant for Khalifa University Library, Abu Dhabi, UAE.**  
> Built in 8 days as part of the Building AI Application Challenge.  
> Live at: **[ku-library.github.io/LibBee](https://ku-library.github.io/LibBee)**

---

## 📌 Overview

LibBee is a production-grade, institution-aware AI chatbot that helps students and researchers at Khalifa University navigate library resources, find academic literature, check live library hours, discover upcoming events, and connect with the right librarian — all through natural language conversation.

Unlike generic chatbots, LibBee is grounded in KU Library's own knowledge base — answering with KU-specific policies, staff, services, and databases. It cannot hallucinate KU-specific facts.

---

## ✨ Key Features

- 💬 **Natural language chat** — ask anything about KU Library
- 🔍 **AI-powered academic search** — LLM builds boolean queries for PRIMO Library Discovery
- ⏰ **Live library hours & events** — real-time data from LibCal API
- 👤 **Staff directory** — name and role-based contact lookup
- 🗄️ **Database recommendations** — subject-specific database guidance
- 📚 **Hybrid RAG** — FAISS dense + BM25 sparse search with RRF fusion
- 🤖 **Dual LLM support** — toggle between GPT-4o and Claude
- 📊 **Admin dashboard** — real-time analytics, metrics, and configuration
- 💰 **Zero infrastructure cost** — entirely on free tiers

---

## 🏗️ Architecture

```
User (GitHub Pages Frontend)
         ↓
FastAPI Backend (HuggingFace Spaces · Port 7860)
         ↓
┌────────────────────────────────────────┐
│           Agent Pipeline               │
│  Rule-based pre-classifier             │
│  → GPT-4o-mini intent classifier       │
│  → RAG / Search / LibCal / LLM         │
└────────────────────────────────────────┘
         ↓
Cloudflare Worker
├── LibCal API Proxy (live hours/events)
└── D1 Analytics (persistent query logs)
```

**Tech Stack:**

| Layer | Technology |
|---|---|
| Frontend | Vanilla JS + HTML (GitHub Pages) |
| Backend | FastAPI + Python 3.9 (HuggingFace Spaces) |
| Classification LLM | GPT-4o-mini (forced, consistent routing) |
| Answering LLM | GPT-4o / Claude Haiku (user toggle) |
| Vector Search | FAISS + OpenAI text-embedding-3-small |
| Keyword Search | BM25 (rank-bm25) |
| Fusion | Reciprocal Rank Fusion (RRF) |
| Live Data | LibCal API via Cloudflare Worker proxy |
| Analytics | Cloudflare D1 (SQLite) |
| Discovery | PRIMO ExLibris API |

---

## 🚀 Quick Start

### Prerequisites
- Python 3.9+
- OpenAI API key
- Anthropic API key (optional — for Claude toggle)
- PRIMO API key (optional — for academic search)
- Cloudflare Worker deployed (for LibCal proxy + analytics)

### 1. Clone the repository

```bash
git clone https://github.com/ku-library/LibBee
cd LibBee
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Set environment variables

Create a `.env` file or set secrets in HuggingFace Spaces:

```env
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key
PRIMO_API_KEY=your_primo_key
ADMIN_PASSWORD=your_admin_password
CLOUDFLARE_WORKER_URL=https://your-worker.workers.dev
SESSION_SECRET=your_session_secret
```

### 4. Run locally

```bash
uvicorn app:app --host 0.0.0.0 --port 7860 --reload
```

### 5. Deploy to HuggingFace Spaces

Push to your HuggingFace Space repository. The `Dockerfile` handles everything automatically.

```bash
git remote add hf https://huggingface.co/spaces/YOUR_USERNAME/LibBee
git push hf main
```

---

## 📁 Project Structure

```
LibBee/
├── app.py                          # FastAPI entry point
├── Dockerfile                      # HuggingFace Spaces deployment
├── requirements.txt                # Python dependencies
├── worker.js                       # Cloudflare Worker (LibCal proxy + D1)
├── index.html                      # Frontend (deploy to GitHub Pages)
├── src/
│   ├── api/
│   │   ├── agent.py                # Main agent — 16-section, 3,900+ lines
│   │   ├── search.py               # PRIMO + PubMed search endpoints
│   │   ├── admin.py                # Admin dashboard API
│   │   └── feedback.py             # Thumbs up/down feedback
│   ├── services/
│   │   ├── rag_service.py          # FAISS + BM25 hybrid RAG
│   │   ├── staff_service.py        # Staff directory matching
│   │   ├── metrics_service.py      # In-memory metrics + async flush
│   │   ├── runtime_store.py        # Atomic JSON config store
│   │   └── cache_service.py        # TTL cache
│   └── config.py                   # Pydantic settings
└── data/
    └── knowledge/                  # 16 KB files (library knowledge base)
        ├── kb_01_general.txt
        ├── kb_02_borrowing.txt
        └── ...
```

---

## 🧠 How Query Routing Works

LibBee routes every query through 8 layers before invoking an LLM:

```
1. Empty / too short         → Prompt for input
2. Pure greeting             → Welcome message (no LLM)
3. Staff name match          → Staff card (no LLM)
4. Staff role match          → Staff card (no LLM)
5. Library hours (regex)     → Live LibCal data (no LLM)
6. Library events (regex)    → Live LibCal events (no LLM)
7. Campus / location         → Hardcoded answer (no LLM)
8. Follow-up detection       → Context-aware response
         ↓
9. LLM Classification (GPT-4o-mini)
         ↓
   library_info → RAG hybrid search
   search_academic → Boolean builder + PRIMO
   search_medical → PubMed + PRIMO + medical contact
   social → Casual LLM response
   general → LLM answer
```

**Result:** Common queries answered instantly at zero LLM cost. LLM invoked only when necessary.

---

## 📊 Evaluation Results

Tested on 40 real-world queries across 7 intent categories:

| Metric | Result |
|---|---|
| Intent classification accuracy | 100% (40/40) |
| Correct routing | 100% (40/40) |
| Live LibCal data accuracy | 100% (5/5) |
| Staff lookup precision | 100% (4/4) — zero false positives |
| Average response time (warm) | ~2.1 seconds |
| Average response time (cold start) | ~65 seconds (FAISS rebuild) |

---

## 💰 Cost Profile

| Component | Cost |
|---|---|
| HuggingFace Spaces (CPU) | $0/month (free tier) |
| GitHub Pages | $0/month |
| Cloudflare Workers + D1 | $0/month (free tier) |
| OpenAI API (GPT-4o-mini) | ~$2–15/month at 500 queries/day |
| Anthropic API (Claude Haiku) | ~$1–8/month at 500 queries/day |
| **Total infrastructure** | **$0 + LLM API costs only** |

---

## 🔧 Admin Dashboard

Access at `/admin` with your configured password.

Features:
- 📊 Real-time Cloudflare D1 analytics (persistent across restarts)
- 📋 Recent query log with intent, tool, model, and response time
- ⚙️ Bot configuration — welcome message, custom instructions, announcements
- 🔄 FAISS + BM25 index rebuild
- ✅ System status — all API connections
- 🔒 Maintenance mode toggle

---

## 🌍 Adapting LibBee for Your Library

LibBee is designed to be replicable. To deploy for your own institution:

1. **Replace the knowledge base** — edit or replace the `.txt` files in `data/knowledge/` with your library's policies, services, and staff
2. **Update staff directory** — edit `STAFF_DIRECTORY` in `src/services/staff_service.py`
3. **Update PRIMO credentials** — set your institution's `PRIMO_API_KEY` and VID
4. **Update LibCal credentials** — configure your LibCal OAuth in the Cloudflare Worker
5. **Update branding** — replace the mascot image and update colours in `index.html`
6. **Deploy** — push to HuggingFace Spaces and GitHub Pages

Any library can be up and running in under a day.

---

## 🤝 Contributing & Collaboration

LibBee is open source and welcomes collaboration from the library and AI communities.

**Areas open for contribution:**
- Arabic language support
- Additional KB content and knowledge domains
- Automated testing suite
- Integration with other library management systems (Ex Libris, Koha, Sierra)
- Mobile app wrapper
- Multi-institution deployment support

**Get in touch:**
- 📧 nikesh.narayanan@ku.ac.ae
- 🔗 [Khalifa University Library](https://library.ku.ac.ae)
- 💬 Open an issue or pull request on GitHub

---

## 📄 License

MIT License — free to use, adapt, and deploy for any library or institution.

---

## 🙏 Acknowledgements

Built as part of the **Building AI Application Challenge 2026**.

Special thanks to the open source community behind FastAPI, LangChain, FAISS, rank-bm25, HuggingFace, and Cloudflare Workers — without which this project would not have been possible.

---

<div align="center">

**🐝 LibBee — Built for every library in the world**

[Try LibBee](https://ku-library.github.io/LibBee) · [GitHub](https://github.com/ku-library/LibBee) · [HuggingFace Space](https://huggingface.co/spaces/nikeshn/LibBee)

*Built with ❤️ by Nikesh Narayanan · Khalifa University Library · Abu Dhabi, UAE*

</div>
