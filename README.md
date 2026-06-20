# lemonick
A local LLM which runs on laptops


Good — this is exactly the right way to think about it. You’re essentially defining a **two-stage product evolution**:

* **Phase 1: Local AI Chat OS (no documents)**
* **Phase 2: Personal Knowledge AI (PDF + learning layer)**

This avoids overbuilding and gets you to a usable product fast.

---

# PRODUCT: Personal AI Server (PAS)

A local-first AI system that runs on laptops via `http://localhost:8080`.

---

# PHASE 1 — CORE LLM APP (NO PDF, NO KNOWLEDGE SYSTEM)

## Goal

> “Install → choose model → chat instantly”

This phase is just a **local ChatGPT replacement**.

---

## Architecture (Phase 1)

```text id="p1_arch"
Browser UI
     │
     ▼
Local API Server
     │
     ├── Model Manager
     ├── Inference Engine (llama.cpp / MLX)
     ├── Download Manager
     ├── Hardware Detector
     ├── Settings Store (SQLite)
     └── Scheduler (idle detection)
```

No embeddings. No vector DB. No documents.

---

# MODULES (PHASE 1 ONLY)

---

## 1. Bootstrap Service

### Purpose

Starts everything cleanly.

### Responsibilities

* Detect OS + hardware
* Start backend server
* Auto-open browser
* Check installed models

### APIs

```http
GET /health
GET /hardware
POST /start
POST /shutdown
```

### Functions

```go
DetectHardware()
StartServer()
OpenBrowser()
ValidateInstall()
```

---

## 2. Hardware Detector

### Purpose

Decide what model the laptop can handle.

### Output Example

```json
{
  "ram_gb": 16,
  "cpu": "Apple M4",
  "gpu": "Metal",
  "recommended": "medium"
}
```

### Functions

```go
GetRAM()
GetCPU()
GetGPU()
BenchmarkInferenceSpeed()
RecommendModel()
```

---

## 3. Model Manager

### Purpose

Handles everything about models.

### Responsibilities

* List available models
* Download models
* Switch models
* Load/unload models

### APIs

```http
GET /models
POST /models/install
POST /models/switch
GET /models/status
DELETE /models/remove
```

### Functions

```go
ListModels()
InstallModel(id)
SwitchModel(id)
LoadModel(path)
UnloadModel()
VerifyChecksum()
```

---

## 4. Download Manager

### Purpose

BitTorrent-like model downloads (resumable, fast, verified).

### APIs

```http
POST /download
GET /download/status
POST /download/pause
POST /download/resume
```

### Functions

```go
StartDownload()
PauseDownload()
ResumeDownload()
VerifySHA256()
ExtractModel()
```

---

## 5. Inference Engine

### Purpose

Runs the LLM locally.

Backends:

* llama.cpp (GGUF)
* MLX (Apple Silicon)

### APIs

```http
POST /chat
POST /generate
POST /stop
```

### Functions

```go
LoadModel()
ChatCompletion()
StreamTokens()
Tokenize()
Detokenize()
Interrupt()
```

---

## 6. Chat Service

### Purpose

Conversation management layer.

### APIs

```http
GET /chat
POST /chat
DELETE /chat/{id}
```

### Functions

```go
CreateChat()
AppendMessage()
LoadHistory()
SummarizeChat()
ExportChat()
```

---

## 7. Settings + Storage

### Purpose

Persist everything locally.

Uses SQLite.

### Tables

```sql
models
settings
chat_history
downloads
usage_stats
```

### APIs

```http
GET /settings
POST /settings
```

### Functions

```go
GetSetting()
SetSetting()
SaveState()
LoadState()
```

---

## 8. Scheduler (Lightweight)

### Purpose

Pause/resume model based on system load.

### Logic

* CPU usage > threshold → pause inference
* user activity detected → reduce priority
* idle → resume full performance

### Functions

```go
DetectIdle()
ThrottleInference()
ResumeInference()
SetPriorityLow()
```

---

## 9. Installer

Native installer for Windows / Mac / Linux
___

# PHASE 1 USER FLOW

```text id="p1_flow"
Install App
     ↓
Detect Hardware
     ↓
Recommend Model
     ↓
Download Model
     ↓
Start Local Server
     ↓
Open Browser UI
     ↓
Chat Immediately
```

That’s it.

---

# PHASE 2 — KNOWLEDGE LAYER (PDF + LEARNING SYSTEM)

Only after Phase 1 is stable.

---

## Goal

> “The AI remembers your documents forever”

---

## Architecture (Phase 2)

```text id="p2_arch"
Browser UI
     │
Local API Server
     │
     ├── Phase 1 (LLM Core)
     ├── Document Ingestion
     ├── Chunking Engine
     ├── Embedding Engine
     ├── Vector DB
     ├── Retrieval Engine
     └── Optional Training Manager
```

---

## MODULES ADDED IN PHASE 2

---

## 9. Document Manager

```http
POST /documents/add
GET /documents
DELETE /documents/{id}
```

Functions:

```go
ImportFile()
StoreMetadata()
TrackVersions()
```

---

## 10. Chunking Engine

Splits long docs into semantic blocks.

```go
SplitText()
CreateOverlapChunks()
NormalizeText()
```

---

## 11. Embedding Engine

Creates vector representations.

```http
POST /embed
```

Functions:

```go
GenerateEmbedding()
BatchEmbed()
```

---

## 12. Vector Database

Search memory.

```http
POST /search
```

Functions:

```go
InsertVector()
QuerySimilar()
DeleteVector()
```

---

## 13. Retrieval Layer

Combines:

* user query
* relevant chunks
* LLM response

```go
RetrieveContext()
RankResults()
BuildPrompt()
```

---

## 14. (Optional later) Training Manager

Only if you still want learning.

```go
CreateDataset()
FineTuneLoRA()
Checkpoint()
MergeAdapter()
```

---

# PHASE 2 USER FLOW

```text id="p2_flow"
Import PDF
     ↓
Chunking
     ↓
Embedding
     ↓
Store in Vector DB
     ↓
Ask question
     ↓
Retrieve context
     ↓
LLM answers with sources
```

---

# KEY DESIGN DECISION (VERY IMPORTANT)

### Phase 1:

* LLM is static
* No memory except chat history
* Focus: performance + UX

### Phase 2:

* Knowledge system added
* Retrieval (not training first)
* Training is optional later

---

# WHAT YOU ARE ACTUALLY BUILDING

This is not a chatbot.

It is:

> **A local AI operating system with optional memory expansion**

---

# If I simplify it even more:

## Phase 1 = "ChatGPT for your laptop"

## Phase 2 = "ChatGPT that remembers your life"

---

If you want next step, I can:

1. Design the **exact folder structure + Go backend skeleton**
2. OR design the **React UI screens**
3. OR start writing **Phase 1 code (boot + model manager first)**


