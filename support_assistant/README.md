# Module 3: Zepto Support Assistant

## Run locally

From this directory, install dependencies and run the notebook cells in order. `MOCK_LLM` is intentionally unset; unset or `1` selects the deterministic, offline graded path. The final Task 5 cell exposes the in-memory FastAPI application object as `app`.

```text
python -m pip install -r requirements.txt
```

Example calls made with the default mock mode:

```powershell
Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8000/ask -ContentType 'application/json' -Body '{"query":"How much is priority delivery?"}'
```

```json
{"answer":"Based on the retrieved context: Zepto delivers grocery and household essentials to serviceable pin codes within 10 to 30 minutes of order confirmation, depending on the customer's delivery zone and current order volume. Standard delivery is free on orders over INR 149; orders below this threshold incur a flat INR 25 delivery fee. Priority delivery, which reserves the next available rider slot, is available at checkout for an additional INR 15.","sources":["doc_01","doc_05","doc_04"],"confidence":1.0}
```

```powershell
Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8000/ask -ContentType 'application/json' -Body '{"query":"What is the capital of France?"}'
```

```json
{"answer":"I can only answer questions about Zepto policies right now.","sources":[],"confidence":1.0}
```

## Architecture

1. **Ingestion:** the setup cell's `get_collection` function reads the eight exact text files in `docs/`. Each document is one chunk, identified by its filename stem (`doc_01` through `doc_08`).
2. **Embedding:** `LocalEmbeddingFunction` uses the local `sentence-transformers` `all-MiniLM-L6-v2` model. ChromaDB stores the vectors in the persistent `zepto_policy_corpus` collection under `chroma_db/`.
3. **Retrieval:** `retrieve_and_answer` embeds the incoming policy query through ChromaDB and retrieves the top three chunks using the collection's cosine-space index.
4. **Generation:** `retrieve_and_answer` creates the grounded answer from the top chunk in mock mode. The optional real path uses `STRUCTURED_PROMPT_TEMPLATE`, which contains the role, context, task, format, length, negative constraint, and few-shot example. `direct_answer` handles general questions without retrieval.

The LangGraph flow is `classify_intent -> retrieve_and_answer` for policy keywords, or `classify_intent -> direct_answer` otherwise. With `MOCK_LLM` unset or `1`, classification uses the required keyword heuristic and both answer nodes use fixed deterministic responses; no network LLM call occurs. With `MOCK_LLM=0`, classification and generation use the optional Groq integration. Real generated JSON is validated against the Pydantic response model and retried up to two additional times with a corrective instruction.

## Docker

The required local container can be built and run as follows:

```powershell
docker build -t zepto-support .
docker run --rm -p 7860:7860 zepto-support
```

Then send the same `POST /ask` requests to `http://127.0.0.1:7860/ask`. No cloud deployment or real LLM key is required.