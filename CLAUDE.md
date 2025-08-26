# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Starting the Application:**
```bash
# Quick start (recommended)
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

**Package Management:**
```bash
# Install dependencies
uv sync

# Add new dependency
uv add <package-name>
```

**Environment Setup:**
```bash
# Copy environment template
cp .env.example .env
# Then edit .env with your ANTHROPIC_API_KEY
```

## Architecture Overview

This is a **RAG (Retrieval-Augmented Generation) System** for querying course materials using semantic search and AI generation.

### Core Architecture Pattern

The system follows a **layered architecture with tool-based AI interaction**:

1. **Frontend (HTML/CSS/JS)** → 2. **FastAPI REST API** → 3. **RAG Orchestrator** → 4. **AI Generator with Tools** → 5. **Vector Store**

### Key Components

**RAG System (`rag_system.py`)**
- Main orchestrator that coordinates all components
- Manages document loading, query processing, and response generation
- Handles incremental document loading (avoids re-processing existing courses)

**Document Processing Pipeline (`document_processor.py`)**
- Parses structured course documents with expected format:
  ```
  Course Title: [title]
  Course Link: [url] 
  Course Instructor: [instructor]
  
  Lesson 0: Introduction
  Lesson Link: [lesson_url]
  [content...]
  ```
- Implements sentence-based chunking with configurable overlap
- Enhances chunks with contextual metadata (course title, lesson number)

**AI Generator (`ai_generator.py`)**
- Manages Claude API interactions with tool calling
- Implements two-phase generation: initial query → tool execution → final response
- Uses predefined system prompts optimized for course content

**Tool System (`search_tools.py`)**
- Extensible tool architecture where Claude autonomously decides when to search
- `CourseSearchTool` supports fuzzy course name matching and lesson filtering
- `ToolManager` handles tool registration and execution coordination

**Vector Store (`vector_store.py`)**
- ChromaDB-based semantic search with two collections:
  - `course_catalog`: Course metadata (titles, instructors)
  - `course_content`: Chunked content with embeddings
- Unified search interface with course name resolution and metadata filtering

**Session Management (`session_manager.py`)**
- Maintains conversation history (configurable max exchanges)
- Provides formatted context for AI continuity

### Configuration (`config.py`)

Key settings in the `Config` dataclass:
- `CHUNK_SIZE`: 800 characters (text chunk size)
- `CHUNK_OVERLAP`: 100 characters (overlap between chunks)
- `MAX_RESULTS`: 5 (search result limit)
- `MAX_HISTORY`: 2 (conversation exchanges to remember)
- `ANTHROPIC_MODEL`: "claude-sonnet-4-20250514"
- `EMBEDDING_MODEL`: "all-MiniLM-L6-v2"

### Data Flow for User Queries

1. **Frontend** sends POST to `/api/query` with query and optional session_id
2. **FastAPI** calls `rag_system.query()`
3. **RAG System** builds prompt and calls AI generator with available tools
4. **Claude** autonomously decides whether to search or use general knowledge
5. **If searching**: Tool system executes semantic search in vector store
6. **Claude** generates final response using search results or general knowledge
7. **Session Manager** updates conversation history
8. **Response** returns with answer and source citations

### File Structure Notes

- `docs/` contains course materials (course1-4_script.txt)
- `frontend/` is a simple HTML/CSS/JS interface
- `backend/` contains all Python components
- ChromaDB data stored in `./chroma_db/` (created automatically)

## Important Implementation Details

**Document Loading:**
- On startup, system loads all `.txt`/`.pdf`/`.docx` files from `docs/`
- Duplicate detection prevents re-processing existing courses
- Use `clear_existing=True` in `add_course_folder()` for full rebuild

**Tool-Based Search:**
- Claude can search by content, course name (fuzzy matching), or specific lesson
- Search results include course/lesson context for accurate attribution
- Sources are automatically tracked and returned to frontend

**Session Continuity:**
- Sessions persist conversation context across multiple queries
- History is truncated to prevent token limit issues
- Frontend maintains session_id across page refreshes
- always use uv to run the server do not use pip directly
- make sure to use uv to manage all dependencies
- use uv to run Python files