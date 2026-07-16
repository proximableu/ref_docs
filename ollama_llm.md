Here is a high-density, context-ready **Ollama Python SDK Reference Sheet** (`ollama_llms.txt`).

---

# Copy-Paste System Reference: Ollama Python SDK

You are writing Python code using the official **Ollama Python library** (`pip install -U ollama`). Adhere strictly to the following framework constraints, response contracts, and API structures.

---

## 1. Client vs. Module API

Ollama can be accessed either via direct top-level functional imports (which use a default global client pointing to `http://localhost:11434`) or by instantiating explicitly configured clients.

### A. The Direct Module API (Sync)

Best for rapid scripting against a local default instance.

```python
from ollama import chat, ChatResponse

# Returns a strongly typed ChatResponse object
response: ChatResponse = chat(
    model='qwen3-coder:30b', 
    messages=[{'role': 'user', 'content': 'Write a Python binary search.'}]
)

# Access content cleanly:
print(response.message.content) 
# Or subscript access: print(response['message']['content'])

```

### B. Instantiated Clients (Sync & Async)

Always instantiate a client explicitly when connecting to custom hosts, adding custom headers, or using async loops.

```python
import asyncio
from ollama import Client, AsyncClient

# 1. Custom Sync Client (All extra kwargs go directly into httpx.Client)
client = Client(
    host='http://127.0.0.1:11434',
    headers={'X-Custom-Header': 'value'}
)
resp = client.chat(model='llama3.3', messages=[{'role': 'user', 'content': 'Hi'}])

# 2. Async Client
async def main():
    async_client = AsyncClient(host='http://127.0.0.1:11434')
    response = await async_client.chat(
        model='llama3.3',
        messages=[{'role': 'user', 'content': 'Hello'}]
    )
    print(response.message.content)

asyncio.run(main())

```

---

## 2. Streaming Responses

Streaming works by setting `stream=True`. It returns a generator yielding chunks of response.

### A. Sync Streaming

```python
from ollama import chat

stream = chat(
    model='qwen3-coder:30b',
    messages=[{'role': 'user', 'content': 'Explain quantum physics.'}],
    stream=True,
)

for chunk in stream:
    # Use flush=True for immediate terminal feedback
    print(chunk.message.content, end='', flush=True)

```

### B. Async Streaming

```python
import asyncio
from ollama import AsyncClient

async def async_stream():
    client = AsyncClient()
    # Note the 'async for ... in await' pattern
    async for chunk in await client.chat(
        model='qwen3-coder:30b',
        messages=[{'role': 'user', 'content': 'Count to 10.'}],
        stream=True
    ):
        print(chunk.message.content, end='', flush=True)

asyncio.run(async_stream())

```

---

## 3. Structured Outputs (Logit Masking / GBNF)

Ollama enforces strict structural outputs via JSON Schemas, using Grammar-Based Sampling at the engine layer (preventing conversational drift). Pydantic is recommended for defining and validating these structures.

### Standard Schema Enforcement Design Pattern:

```python
from ollama import chat
from pydantic import BaseModel, Field

# 1. Define your strict Pydantic Schema
class DependencyInfo(BaseModel):
    package_name: str
    license: str
    verdict: str = Field(description="Must be 'SAFE', 'REVIEW', or 'FORBIDDEN'")
    risk_score: int = Field(ge=0, le=10)

class AuditResult(BaseModel):
    dependencies: list[DependencyInfo]

# 2. Generate and validate the response
response = chat(
    model='qwen3-coder:30b',
    messages=[{
        'role': 'user', 
        'content': 'Analyze pip packages: fastapi, requests, and cryptography.'
    }],
    # Extract the schema directly using Pydantic's model_json_schema()
    format=AuditResult.model_json_schema(),
    # Set temperature to 0 for highly deterministic structuring
    options={'temperature': 0} 
)

# 3. Parse and validate safely inside Python types
try:
    audit = AuditResult.model_validate_json(response.message.content)
    for dep in audit.dependencies:
        print(f"{dep.package_name} -> {dep.verdict} (Risk: {dep.risk_score}/10)")
except Exception as e:
    print(f"Validation failed: {e}")

```

---

## 4. Multi-Modal Vision Calls

Use `llama3.2-vision` or another multi-modal model. Image files can be passed as raw bytes or paths.

```python
from ollama import chat

response = chat(
    model='llama3.2-vision',
    messages=[{
        'role': 'user',
        'content': 'Describe what you see in this screenshot.',
        # List of image filepaths (or file bytes)
        'images': ['./workspace_bug.png'] 
    }]
)
print(response.message.content)

```

---

## 5. Embeddings API

Obtain dense mathematical vectors for RAG implementations.

```python
import ollama

# Single Vector
embed_single = ollama.embed(
    model='nomic-embed-text',
    input='The quick brown fox jumps over the lazy dog.'
)
vector = embed_single.embeddings[0]

# Batch Embeddings (highly optimized)
embed_batch = ollama.embed(
    model='nomic-embed-text',
    input=[
        'Query text for matching database records.',
        'Document content stored inside the vector index.'
    ]
)
# Returns a list of vectors: embed_batch.embeddings[0], embed_batch.embeddings[1]

```

---

## 6. Model Management API

| Operation | Synchronous Syntax | Notes |
| --- | --- | --- |
| **Pull Model** | `ollama.pull('gemma3')` | Downloads a model from the Ollama registry |
| **List Models** | `ollama.list()` | Returns metadata of local models (`.models`) |
| **Active Models** | `ollama.ps()` | Returns list of currently running/loaded models |
| **Show Model** | `ollama.show('llama3')` | Returns parameter sizes, system prompt, and layout details |
| **Create Custom Model** | `ollama.create(model='my-agent', from_='llama3', system='Prompt')` | Programmatic equivalent to writing a `Modelfile` |
| **Copy Model** | `ollama.copy('llama3', 'custom-backup')` | Clones a locally stored model |
| **Delete Model** | `ollama.delete('old-model')` | Deletes a model from disk to free up GPU memory |

### Example: Checking Running Models (To manage VRAM overhead)

```python
import ollama

running = ollama.ps()
for model in running.models:
    print(f"Running: {model.model} (VRAM: {model.size_vram / 1e9:.2f} GB)")

```

---

## 7. Error Handling

```python
import ollama

try:
    ollama.chat(model='non-existent-model', messages=[{'role': 'user', 'content': 'Hi'}])
except ollama.ResponseError as e:
    print(f"Ollama API Error (Code {e.status_code}): {e.error}")
except Exception as e:
    print(f"Connection or unexpected error: {e}")

```