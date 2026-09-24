# Support Assistant

## Objective

This module implements the Zepto policy support assistant described in the capstone. The intended architecture is:

```text
Policy documents
      |
      v
Ingestion / chunking
      |
      v
Sentence-Transformers
all-MiniLM-L6-v2
      |
      v
ChromaDB (cosine similarity)
      |
      v
LangGraph intent router
      |
      +-----------------------------+
      |                             |
policy_question               general_question
      |                             |
      v                             v
retrieve top-3                  direct answer
      |
      v
mock or real generation
      |
      v
Pydantic RAGResponse
      |
      v
FastAPI POST /ask
```

The project specification defines mock mode as the graded baseline: `MOCK_LLM` is unset or set to `1`, and no external LLM call is required.

## Supplied Files

The supplied archive currently contains:

```text
support-assistant/
├── Documents.txt
├── support_assistant.py
└── .env
```

The assignment, however, expects the module directory to be named:

```text
support_assistant/
```

and requires **8 separate corpus files**. Before submission, reorganize the supplied policy text into eight files such as:

```text
docs/
├── doc_01.txt
├── doc_02.txt
├── doc_03.txt
├── doc_04.txt
├── doc_05.txt
├── doc_06.txt
├── doc_07.txt
└── doc_08.txt
```

Use the exact corpus wording provided by the project description.

## Architecture

### 1. Ingestion

The current application reads `Documents.txt` line-by-line and creates one non-empty line as one chunk. Each chunk receives an ID such as `doc_01`, `doc_02`, etc.

### 2. Embedding

`SentenceTransformer('all-MiniLM-L6-v2')` generates local embeddings. This avoids requiring an API key for the graded path.

### 3. Retrieval

A ChromaDB collection named:

```text
policy_documents
```

is created with cosine similarity. For policy questions, the query is embedded and the top three chunks are retrieved.

### 4. Generation

The LangGraph graph contains:

- `classify_intent`
- `retrieve_and_answer`
- `direct_answer`

In mock mode, `retrieve_and_answer` creates:

```text
Based on the retrieved context: <top retrieved chunk excerpt>
```

and `direct_answer` returns a fixed out-of-domain response.

When `MOCK_LLM=0`, the policy retrieval path uses the structured prompt and optional Groq-backed generation, while the general-question path can call the configured LLM directly.

### 5. Structured Output

The Pydantic response model is:

```python
class RAGResponse(BaseModel):
    answer: str
    sources: List[str]
    confidence: float
```

This provides the required structured response fields.

### 6. FastAPI

The application exposes:

```text
POST /ask
```

with a request body containing:

```json
{"query": "your question"}
```

and returns a validated `RAGResponse`.

## Prompt Design

The supplied code includes a structured prompt containing all five requested components:

- **Role**
- **Context**
- **Task**
- **Format**
- **Length**

It also contains explicit negative constraints requiring answers to rely only on retrieved context and includes two few-shot examples.

The prompt is used by the optional real-LLM policy-answer path.

## Mock Mode

The required baseline is deterministic and offline:

```bash
MOCK_LLM=1
```

or simply leave `MOCK_LLM` unset.

The mock intent classifier looks for:

```text
delivery
return
refund
membership
tracking
cancel
gift card
support hours
```

A query containing one of these terms is routed to `policy_question`; other queries are routed to `general_question`.

For policy questions, embeddings and ChromaDB retrieval still run. Only the final generation step is mocked.

## Example Requests

After the application has been corrected and started with mock mode enabled, the README required by the assignment should record the **actual raw JSON responses** from two calls.

Example policy-style request:

```json
{
  "query": "What is the delivery fee for an order below INR 149?"
}
```

Expected response structure:

```json
{
  "answer": "Based on the retrieved context: ...",
  "sources": ["doc_01", "..."],
  "confidence": 1.0
}
```

Example general request:

```json
{
  "query": "What is the capital of India?"
}
```

Expected response structure in mock mode:

```json
{
  "answer": "Out of context,answers only about Zepto policies right now.",
  "sources": [],
  "confidence": 1.0
}
```

These are response structures/examples, not substitutes for running the service and recording the actual responses required by the assignment.

## Running Locally

After renaming the folder to `support_assistant` and correcting the syntax issue described below:

```bash
cd support_assistant
pip install sentence-transformers chromadb langgraph fastapi uvicorn pydantic python-dotenv groq

# Windows
set MOCK_LLM=1

# macOS/Linux
export MOCK_LLM=1

uvicorn support_assistant:app --host 0.0.0.0 --port 7860
```

Test with:

```bash
curl -X POST http://127.0.0.1:7860/ask   -H "Content-Type: application/json"   -d '{"query":"What is the delivery fee for an order below INR 149?"}'
```

and:

```bash
curl -X POST http://127.0.0.1:7860/ask   -H "Content-Type: application/json"   -d '{"query":"What is the capital of India?"}'
```

## Docker

The assignment requires a locally buildable and runnable Dockerfile. The supplied archive does **not** currently contain a Dockerfile, so one must be added before submission.

A suitable baseline command sequence after adding a Dockerfile is:

```bash
docker build -t zepto-support-assistant .
docker run --rm -p 7860:7860 -e MOCK_LLM=1 zepto-support-assistant
```

The container should expose the FastAPI application on port `7860`.

## Current Implementation Issues to Fix Before Submission

The supplied code was reviewed directly against the project specification. The following items should be addressed:

1. **Syntax error:** `classify_intent()` ends with `return {"intent": intent}s`, which is invalid Python. It should return the dictionary without the trailing `s`.
2. **Folder name:** the supplied archive uses `support-assistant`; the assignment requires `support_assistant`.
3. **Eight corpus files:** the supplied archive contains one `Documents.txt` file rather than eight separate corpus files.
4. **File path:** the application first looks for `support-assistant/Documents.txt`. If the module is renamed to `support_assistant`, that path should be updated or simplified.
5. **Dockerfile:** none is present in the supplied archive; the assignment requires one.
6. **Real-LLM intent branch:** the project description says the optional `MOCK_LLM=0` mode may use an LLM for intent classification. The current code continues using the keyword heuristic in that branch. This is acceptable only if the optional extension is not being claimed as implemented.
7. **Real-LLM general response:** the general-question real-LLM path returns a response with `confidence=0.8` and does not apply the same structured prompt used by the policy path. Review this if implementing the optional extension.
8. **Recorded API examples:** actual raw JSON responses from both required example calls should be run and copied into this README before submission.
9. **`.env`:** do not commit an API key or secret. If the supplied `.env` contains credentials, remove them from the repository and use environment variables or a secret-management mechanism.

## Design Summary

The intended design keeps retrieval independent of the LLM provider: embeddings and ChromaDB are local, while only answer generation changes with `MOCK_LLM`. This makes the default path deterministic and offline while leaving an optional real-LLM extension behind the same application interface.
