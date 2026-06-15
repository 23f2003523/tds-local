# RAG Query API

A FastAPI-based Retrieval-Augmented Generation (RAG) service that answers user questions using a local knowledge base of documentation and discussion forum content. The system performs semantic search using vector embeddings and generates context-aware responses with source citations.

---

## Features

* Semantic search using OpenAI embeddings
* Retrieval-Augmented Generation (RAG)
* Support for documentation and forum/discussion content
* Multimodal queries (text + image)
* Automatic source citation generation
* SQLite-based vector storage
* FastAPI REST API
* Health monitoring endpoint
* Retry and error-handling mechanisms

---

## Architecture

```text
User Query
    │
    ▼
Generate Embedding
    │
    ▼
Similarity Search
(SQLite Knowledge Base)
    │
    ▼
Retrieve Relevant Chunks
    │
    ▼
Context Enrichment
(Adjacent Chunks)
    │
    ▼
LLM Response Generation
    │
    ▼
Answer + Sources
```

---

## Project Structure

```text
.
├── app.py
├── knowledge_base.db
├── .env
├── requirements.txt
└── README.md
```

---

## Database Schema

### discourse_chunks

Stores chunks extracted from discussion forums.

| Column      | Type    |
| ----------- | ------- |
| id          | INTEGER |
| post_id     | INTEGER |
| topic_id    | INTEGER |
| topic_title | TEXT    |
| post_number | INTEGER |
| author      | TEXT    |
| created_at  | TEXT    |
| likes       | INTEGER |
| chunk_index | INTEGER |
| content     | TEXT    |
| url         | TEXT    |
| embedding   | BLOB    |

---

### markdown_chunks

Stores chunks extracted from documentation.

| Column        | Type    |
| ------------- | ------- |
| id            | INTEGER |
| doc_title     | TEXT    |
| original_url  | TEXT    |
| downloaded_at | TEXT    |
| chunk_index   | INTEGER |
| content       | TEXT    |
| embedding     | BLOB    |

---

## Requirements

* Python 3.9+
* OpenAI-compatible API key
* SQLite

---

## Installation

### Clone Repository

```bash
git clone <repository-url>
cd <repository-name>
```

### Create Virtual Environment

```bash
python -m venv venv
```

Activate:

**Linux/macOS**

```bash
source venv/bin/activate
```

**Windows**

```bash
venv\Scripts\activate
```

### Install Dependencies

```bash
pip install -r requirements.txt
```

---

## Environment Variables

Create a `.env` file:

```env
API_KEY=your_api_key_here
```

The application uses this key for:

* Embedding generation
* LLM responses
* Image understanding

---

## Running the Application

### Development

```bash
python app.py
```

or

```bash
uvicorn app:app --reload
```

Server starts on:

```text
http://localhost:8000
```

---

## API Endpoints

### Health Check

#### Request

```http
GET /health
```

#### Response

```json
{
  "status": "healthy",
  "database": "connected",
  "api_key_set": true,
  "discourse_chunks": 1000,
  "markdown_chunks": 500,
  "discourse_embeddings": 1000,
  "markdown_embeddings": 500
}
```

---

### Query Knowledge Base

#### Request

```http
POST /query
Content-Type: application/json
```

##### Text Query

```json
{
  "question": "What are the eligibility criteria?"
}
```

##### Multimodal Query

```json
{
  "question": "What error is shown in this screenshot?",
  "image": "<base64-encoded-image>"
}
```

---

#### Response

```json
{
  "answer": "The eligibility criteria are...",
  "links": [
    {
      "url": "https://example.com/doc",
      "text": "Relevant source snippet"
    }
  ]
}
```

---

## How It Works

### 1. Query Processing

The user's question (and optional image) is processed.

For image-based queries:

* Image is analyzed using GPT-4o-mini vision capabilities.
* Image description is merged with the user's question.

---

### 2. Embedding Generation

The combined query is converted into a vector embedding using:

```text
text-embedding-3-small
```

---

### 3. Similarity Search

The query embedding is compared against stored embeddings using cosine similarity.

```python
similarity = dot(v1, v2) / (||v1|| * ||v2||)
```

Only chunks above the configured threshold are retained.

Default threshold:

```python
SIMILARITY_THRESHOLD = 0.68
```

---

### 4. Context Enrichment

Adjacent chunks are fetched to improve context quality and reduce fragmented answers.

---

### 5. Answer Generation

Relevant chunks are provided to:

```text
gpt-4o-mini
```

The model is instructed to:

* Answer only from retrieved context
* Avoid hallucinations
* Provide source references

---

## Configuration

Adjust retrieval settings in `app.py`:

```python
SIMILARITY_THRESHOLD = 0.68
MAX_RESULTS = 10
MAX_CONTEXT_CHUNKS = 4
```

| Parameter            | Description                |
| -------------------- | -------------------------- |
| SIMILARITY_THRESHOLD | Minimum similarity score   |
| MAX_RESULTS          | Maximum retrieved chunks   |
| MAX_CONTEXT_CHUNKS   | Chunks retained per source |

---

## Error Handling

The API includes:

* Retry mechanisms for API failures
* Exponential backoff for rate limits
* Database connection validation
* Embedding validation
* Graceful fallback for image-processing failures

---

## Example Usage

### cURL

```bash
curl -X POST http://localhost:8000/query \
-H "Content-Type: application/json" \
-d '{
  "question":"What is this knowledge base about?"
}'
```

---

### Python

```python
import requests

response = requests.post(
    "http://localhost:8000/query",
    json={
        "question": "What are the course requirements?"
    }
)

print(response.json())
```

---

## Future Improvements

* FAISS or Chroma vector database integration
* Hybrid search (keyword + vector)
* Streaming responses
* Conversation memory
* Metadata filtering
* Batch embedding generation
* Authentication and rate limiting
* Docker deployment

---

## License

This project is provided as-is for educational and research purposes.

---

## Author

Built using:

* FastAPI
* SQLite
* NumPy
* OpenAI Embeddings
* GPT-4o-mini
* aiohttp
