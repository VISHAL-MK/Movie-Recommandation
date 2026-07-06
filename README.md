CineDNA — Agentic AI Movie Companion

CineDNA is a full-stack agentic AI platform that builds a personalized Movie DNA profile from your taste and uses it to deliver deeply personalized movie recommendations through natural conversation. It runs entirely on local hardware — no cloud AI APIs required.


Think of it as an AI friend who genuinely understands your cinematic taste and talks about movies like a fellow film lover, not a search engine.




How It Works


DNA Profiling — You pick your favorite genres, list movies you love, and share a few taste preferences. CineDNA synthesizes this into a multi-dimensional taste profile (soul archetype, character DNA, hidden tastes).
Conversational AI — Chat with CineDNA naturally. It uses your DNA profile, TMDB movie data, and conversation history to give recommendations that feel personal.
Taste Evolution — As you rate movies and interact, CineDNA tracks how your preferences shift over time.
Discovery — Browse DNA-matched picks, search any movie, or explore what's trending — with full detail views.



Architecture

┌─────────────────────────────────────────────────────┐
│  Frontend (React + Vite)                            │
│  Landing · Movie DNA · Chat · Discover · Profile    │
└──────────────────────┬──────────────────────────────┘
                       │ REST + SSE
┌──────────────────────▼──────────────────────────────┐
│  Backend (FastAPI)                                   │
│                                                      │
│  ┌──────────┐  ┌───────────┐  ┌──────────────────┐  │
│  │ LLM Agent│  │ RAG Layer │  │ MCP Tool Server  │  │
│  │ (Qwen 3) │  │ (ChromaDB)│  │ (TMDB, Profile,  │  │
│  │ LangChain│  │ Embeddings│  │  Taste, Recs)    │  │
│  └──────────┘  └───────────┘  └──────────────────┘  │
│                                                      │
│  ┌──────────────────────────────────────────────┐    │
│  │ SQLite — Users, DNA, Ratings, Chat, Evolution│    │
│  └──────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────┘


Tech Stack

LayerTechnologyFrontendReact 18, Vite 6, React Router, AxiosBackendFastAPI, Uvicorn, PydanticLLMQwen 3 (1.7B) via Ollama — runs locallyAgent FrameworkLangChain with tool-callingRAGChromaDB + Sentence Transformers (all-MiniLM-L6-v2)Movie DataTMDB API (search, details, trending, similar)DatabaseSQLite (persistent — users, DNA, ratings, chat history, taste evolution)StreamingServer-Sent Events (SSE) for real-time token streamingTool ProtocolModel Context Protocol (MCP) server


Project Structure

cinedna-v3/
├── backend/
│   ├── main.py                 # FastAPI app entry point
│   ├── core/config.py          # Environment settings (Pydantic)
│   ├── api/routes/
│   │   ├── chat.py             # Chat + streaming endpoints
│   │   ├── dna.py              # DNA generation & retrieval
│   │   ├── movie.py            # Movie detail lookup
│   │   ├── profile.py          # User profile, ratings, evolution
│   │   └── recommendations.py  # DNA picks, search, trending
│   ├── llm/agent.py            # CineDNA Agent (LangChain + Qwen 3)
│   ├── rag/pipeline.py         # RAG context builder
│   ├── db/database.py          # SQLite schema & queries
│   ├── services/
│   │   ├── tmdb.py             # TMDB API client
│   │   └── dna_profiler.py     # DNA synthesis logic
│   └── mcp_server/
│       ├── server.py           # MCP server entry
│       └── tools/              # Tool modules (TMDB, profile, taste, recs)
├── frontend/
│   ├── src/
│   │   ├── pages/
│   │   │   ├── Landing.jsx     # Home page
│   │   │   ├── MovieDNA.jsx    # DNA setup wizard
│   │   │   ├── Chat.jsx        # Conversational AI interface
│   │   │   ├── Recommendations.jsx  # Discover page (DNA picks, search, trending)
│   │   │   └── Profile.jsx     # User profile & ratings
│   │   ├── components/
│   │   │   ├── Navbar.jsx
│   │   │   ├── MovieCard.jsx
│   │   │   └── MovieDetailModal.jsx
│   │   └── api/client.js       # Axios API client
│   └── package.json
└── README.md


Getting Started

Prerequisites


Node.js ≥ 18
Python ≥ 3.10
Ollama installed and running (ollama.com)
TMDB API key — get one free at themoviedb.org


1. Pull the LLM model

bashollama pull qwen3:1.7b

2. Backend setup

bashcd cinedna-v3/backend

# Create .env file
echo TMDB_API_KEY=your_tmdb_bearer_token_here > .env

# Install dependencies and run
uv sync
uv run uvicorn main:app --reload --port 8000

The backend starts at http://localhost:8000. The SQLite database (cinedna.db) is created automatically on first run.

3. Frontend setup

bashcd cinedna-v3/frontend

npm install
npm run dev

The frontend starts at http://localhost:5173.


API Endpoints

MethodEndpointDescriptionPOST/api/dna/generateGenerate a user's DNA profileGET/api/dna/{user_id}Retrieve DNA profilePOST/api/chatChat with CineDNA (supports SSE streaming)GET/api/chat/history/{user_id}Get chat historyDELETE/api/chat/history/{user_id}Clear chat historyGET/api/recommendations/{user_id}Get DNA-matched recommendationsGET/api/recommendations/search?q=...Search movies by keywordGET/api/recommendations/trending/nowGet trending moviesGET/api/movie/{movie_id}Get full movie detailsGET/api/profile/{user_id}Get user profilePOST/api/profile/{user_id}/rateRate a movieGET/api/profile/{user_id}/evolutionGet taste evolution history


Environment Variables

Create a .env file in the backend/ directory:

envTMDB_API_KEY=your_tmdb_bearer_token
OLLAMA_MODEL=qwen3:1.7b
OLLAMA_BASE_URL=http://localhost:11434
DB_PATH=cinedna.db


Key Design Decisions


Local-first — The entire AI pipeline (LLM, embeddings, vector store) runs on your machine. No OpenAI/Google/Anthropic API keys needed.
Conversational, not transactional — The AI is prompted to speak like a movie-loving friend, not a data retrieval system. No Wikipedia-style structured output.
Agentic tool-calling — The LLM autonomously decides when to search TMDB, look up user profiles, or fetch similar movies. Up to 3 tool calls per conversation turn.
Streaming responses — SSE-based token streaming so the user sees the AI "typing" in real time.
Persistent state — Everything is stored in SQLite: user profiles, DNA, ratings, chat history, taste evolution snapshots.
