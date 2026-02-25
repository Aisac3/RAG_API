## RAG API

This project is a minimal Retrieval-Augmented Generation (RAG) API built with **FastAPI**, **ChromaDB** for vector storage, and **Ollama** for local LLM inference.

You can:
- **Populate a knowledge base** from local text files.
- **Add new knowledge at runtime** via an API.
- **Query the knowledge base** and get LLM-generated answers grounded in stored documents.
- **Explore and test the API** using the built-in **Swagger UI**.

---

### Requirements

- **Python 3.13** (or the version used to create the existing `venv`)
- **Ollama** installed and running, with the `tinyllama` model pulled  
  - Install Ollama from their website.  
  - Then run: `ollama pull tinyllama`

---

### Installation & Setup

From the project root (`my_rag-api`):

```bash
python -m venv venv
source venv/bin/activate          # On macOS / Linux
# .\venv\Scripts\activate         # On Windows PowerShell

pip install fastapi uvicorn chromadb ollama
```

If you already have the `venv` in this repo, just activate it:

```bash
source venv/bin/activate
```

---

### Pre-populating the Knowledge Base

The script `embed.py` loads the content of `k8s.txt` into a persistent Chroma collection named `docs`.

Run:

```bash
source venv/bin/activate
python embed.py
```

This will:
- Create / reuse a Chroma DB under the `db/` directory.
- Store the text from `k8s.txt` as an initial document in the `docs` collection.

You can repeat this pattern for other files if you want to batch-ingest content.

---

### Running the API

Start the FastAPI server with Uvicorn:

```bash
source venv/bin/activate
uvicorn app:app --reload
```

The main endpoints:

- **Health / info**: `GET /`
- **Query KB + LLM**: `POST /query`
- **Add to KB**: `POST /add`
- **Swagger UI**: `GET /docs`
- **OpenAPI JSON**: `GET /openapi.json`

---

### API Usage

#### 1. Swagger UI

Once the server is running, open your browser at:

- `http://127.0.0.1:8000/docs`

From there you can:
- Inspect all endpoints and their schemas.
- Send test requests for `/query` and `/add`.

#### 2. `POST /query`

**Description**:  
Send a natural language question; the API:
- Retrieves the most relevant document from the Chroma `docs` collection.
- Passes the document text + your question as context to the `tinyllama` model via Ollama.
- Returns a concise answer.

**Example (curl)**:

```bash
curl -X POST "http://127.0.0.1:8000/query?q=What is Kubernetes?" 
```

**Response (shape)**:

```json
{
  "answer": "..."
}
```

#### 3. `POST /add`

**Description**:  
Add a new text snippet to the knowledge base.

**Example (curl)**:

```bash
curl -X POST "http://127.0.0.1:8000/add" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "text=Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services."
```

**Response (shape)**:

```json
{
  "status": "success",
  "message": "Content added to knowledge base",
  "id": "generated-uuid"
}
```

The new document is persisted in the same Chroma `docs` collection used by `/query`.

---

### Project Structure

```text
app.py          # FastAPI app: /query and /add endpoints, Swagger UI enabled
embed.py        # Script to load k8s.txt into Chroma DB
k8s.txt         # Example knowledge base seed file
db/             # Persistent ChromaDB storage
venv/           # Python virtual environment (local to this project)
```

---

### Notes & Tips

- **Model**: The app currently uses the `tinyllama` model via Ollama. You can switch models by editing the `model` argument in `app.py`.
- **Persistence**: Chroma is configured with `PersistentClient(path="./db")`, so your KB survives restarts.
- **Experimentation**: Use the Swagger UI (`/docs`) to quickly test and iterate on queries and added knowledge.

