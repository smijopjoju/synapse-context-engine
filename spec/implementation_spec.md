# Implementation Specification: Synapse Context Engine

## 1. Overview
The **Synapse Context Engine** is a local-first project memory solution designed to provide long-term context for AI-assisted development. It stores project-specific instructions, architectural decisions, and "learned" context using a hybrid storage approach (SQLite for text/metadata and LanceDB for vector embeddings). The system is accessible via an **MCP (Model Context Protocol) Server** for AI integration and a **Web UI** for manual management.

### Key Goals
*   **Privacy:** 100% local execution; no data ever leaves the machine.
*   **Persistence:** Long-term memory that survives context window resets.
*   **Human-in-the-loop:** A Web UI to review, edit, and delete memories.
*   **Seamless Integration:** MCP tools for automated memory retrieval and storage during coding sessions.

---

## 2. System Architecture

### 2.1 Technical Stack
*   **Backend:** Python 3.10+ with FastAPI.
*   **Database (Relational):** SQLite (via SQLAlchemy or SQLModel) for source-of-truth text and metadata.
*   **Database (Vector):** LanceDB (serverless, disk-based) for semantic search.
*   **Embeddings:** Ollama (default: `nomic-embed-text`) running locally.
*   **Frontend:** React (Vite) + Tailwind CSS + Shadcn UI.
*   **Interface:** MCP Python SDK for the server implementation.

### 2.2 Component Diagram
```text
[ IDE / AI Assistant ] <--> [ MCP Server (FastAPI) ] <--> [ Ollama (Local Embeddings) ]
                                     ^
                                     |
[ Web UI (React) ] <-----------> [ REST API ]
                                     |
                                     v
                        [ Hybrid Store: SQLite + LanceDB ]
```

---

## 3. Data Model

### 3.1 Project
*   `id`: UUID (Primary Key)
*   `name`: String (Unique)
*   `path`: String (Optional, local filesystem path)
*   `created_at`: Timestamp

### 3.2 Memory (The "Synapse")
*   `id`: UUID (Primary Key)
*   `project_id`: UUID (Foreign Key)
*   `content`: Text (The actual instruction or decision)
*   `embedding_id`: String (Reference to LanceDB vector)
*   `tags`: JSON/Array of strings
*   `metadata`: JSON (e.g., file context, line numbers, git commit)
*   `created_at`: Timestamp
*   `updated_at`: Timestamp

---

## 4. Interfaces

### 4.1 MCP Tools
*   `add_memory(project_name: str, content: str, tags: list[str])`: Creates a new memory entry.
*   `search_memory(project_name: str, query: str, top_k: int = 5)`: Returns semantically relevant memories.
*   `list_memories(project_name: str)`: Returns all memories for a project (paginated).
*   `delete_memory(memory_id: str)`: Removes a specific memory.

### 4.2 Web UI Features
*   **Dashboard:** Overview of all tracked projects.
*   **Memory Manager:** A searchable, filterable list of memories for a selected project.
*   **CRUD Operations:** Directly edit the text of a memory (triggering an embedding update) or delete obsolete entries.
*   **Import/Export:** Export project context to Markdown or JSON.

---

## 5. Implementation Phases

### Phase 1: Core Engine (Storage & Embeddings)
*   Setup SQLite schema.
*   Integrate LanceDB with Ollama client.
*   Implement `SynapseStore` class to handle atomic updates to both DBs.

### Phase 2: MCP Server & REST API
*   Build FastAPI wrapper.
*   Implement MCP lifecycle (SSE or stdio).
*   Expose REST endpoints for the UI.

### Phase 3: Web UI
*   Create a clean, minimalist interface for project/memory management.
*   Implement "Live Search" using both keyword (SQLite) and semantic (LanceDB) filters.

### Phase 4: Integration & Testing
*   Verify end-to-end flow: AI adds memory -> Memory appears in UI -> UI edit memory -> AI retrieves updated memory.

---

## 6. Security & Privacy
*   **Data Locality:** All databases stored in `~/.synapse/data/`.
*   **No Telemetry:** No external pings or analytics.
*   **API Security:** Localhost-only binding for the FastAPI server.
