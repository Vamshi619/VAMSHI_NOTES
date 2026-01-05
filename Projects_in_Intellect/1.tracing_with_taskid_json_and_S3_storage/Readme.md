
# Process Flow: S3 Async Individual Storage Implementation

Let me break down the complete flow with inputs, processing steps, and outputs.

---

## High-Level Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CURRENT FLOW                                    │
│                                                                             │
│  User Request → FastAPI → TaskManager → Local JSON File (tasks.json)        │
│                              ↓                                              │
│                         Background Workers                                  │
│                              ↓                                              │
│                    External AI Platform (PF Interface)                      │
└─────────────────────────────────────────────────────────────────────────────┘

                                    ↓ MIGRATE TO ↓

┌─────────────────────────────────────────────────────────────────────────────┐
│                              NEW FLOW                                        │
│                                                                             │
│  User Request → FastAPI → TaskManager → S3 (Individual JSON files)          │
│                              ↓                                              │
│                         Background Workers (Async)                          │
│                              ↓                                              │
│                    External AI Platform (PF Interface)                      │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Process Flow

### **STEP 1: Task Submission**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: TASK SUBMISSION                                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  INPUT (from User):                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ POST /task/submit                                                    │   │
│  │ {                                                                    │   │
│  │   "username": "user@example.com",                                    │   │
│  │   "password": "xxx",                                                 │   │
│  │   "data": [                                                          │   │
│  │     {"Market Value": "12000000", "Top Asset": "Equity"},             │   │
│  │     {"top_movers_data": "..."},                                      │   │
│  │     {"total_portfolio_mv": 1000000, "assets": [...]}                 │   │
│  │   ]                                                                  │   │
│  │ }                                                                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  PROCESSING:                                                                │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 1. Generate task_id: "task_20251230182721_01e09305"                  │   │
│  │ 2. Create Task object with status=PENDING                           │   │
│  │ 3. Add initial trace entry                                          │   │
│  │ 4. Save to S3: s3://bucket/tasks/task_20251230182721_01e09305.json  │   │
│  │ 5. Add task_id to async queue                                       │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  OUTPUT (to User):                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ {                                                                    │   │
│  │   "task_id": "task_20251230182721_01e09305",                         │   │
│  │   "status": "pending",                                               │   │
│  │   "message": "Task submitted successfully...",                       │   │
│  │   "created_at": "2025-12-30T18:27:21.828027"                         │   │
│  │ }                                                                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  S3 OUTPUT (Individual File):                                               │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ s3://wealth-ai-tasks/tasks/task_20251230182721_01e09305.json         │   │
│  │ {                                                                    │   │
│  │   "task_id": "task_20251230182721_01e09305",                         │   │
│  │   "status": "pending",                                               │   │
│  │   "progress": 0,                                                     │   │
│  │   "username": "...",                                                 │   │
│  │   "data": [...],                                                     │   │
│  │   "trace": [{"step": "Submit", "status": "completed", ...}]          │   │
│  │ }                                                                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### **STEP 2: Background Worker Picks Task**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: WORKER PICKS TASK FROM QUEUE                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  INPUT (from Queue):                                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ task_id = "task_20251230182721_01e09305"                             │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  PROCESSING:                                                                │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 1. Worker calls: await task_queue.get()                              │   │
│  │ 2. Fetch task from S3 (or cache)                                     │   │
│  │ 3. Start processing pipeline                                         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  INTERMEDIATE OUTPUT:                                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ Task object loaded into worker memory                                │   │
│  │ Log: "Worker 0 processing task task_20251230182721_01e09305"         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### **STEP 3: Authentication**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 3: AUTHENTICATION                                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  INPUT:                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ AuthRequest {                                                        │   │
│  │   username: "vamshi.pallapu@intellectdesign.com",                    │   │
│  │   password: "Elephant@_619",                                         │   │
│  │   tenant: "idigi"                                                    │   │
│  │ }                                                                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  PROCESSING:                                                                │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 1. Update task: status=AUTHENTICATING, progress=10                   │   │
│  │ 2. Save to S3 (async)                                                │   │
│  │ 3. Call: InvokeAsset.authenticate(auth_request)                      │   │
│  │ 4. External API call to: GET /accesstoken/{tenant}                   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  INTERMEDIATE OUTPUT:                                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ AuthResponse {                                                       │   │
│  │   access_token: "eyJhbGciOiJSUzI1NiIsInR5cCI...",                    │   │
│  │   token_type: "Bearer"                                               │   │
│  │ }                                                                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  S3 UPDATE:                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ {                                                                    │   │
│  │   "status": "authenticated",                                         │   │
│  │   "progress": 20,                                                    │   │
│  │   "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI...",                  │   │
│  │   "trace": [..., {"step": "1. Authentication", "status": "completed"}]│  │
│  │ }                                                                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### **STEP 4: Section Detection**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 4: SECTION DETECTION                                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  INPUT (from task.data - 3 items):                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ Input 1: {"Market Value", "Investment Value", "Top Asset", ...}      │   │
│  │ Input 2: {"top_movers_data": "..."}                                  │   │
│  │ Input 3: {"total_portfolio_mv", "customer_risk_profile", "assets"}   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  PROCESSING (for each input):                                               │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 1. Load config.json                                                  │   │
│  │ 2. Match input fields against section required_fields                │   │
│  │ 3. Return (section_name, asset_id) for each input                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  INTERMEDIATE OUTPUT:                                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ sections_to_process = [                                              │   │
│  │   {                                                                  │   │
│  │     "index": 0,                                                      │   │
│  │     "section_name": "portfolio_snapshot",                            │   │
│  │     "asset_id": "c7b75809-063a-4f9c-80a8-496e88ed4894",              │   │
│  │     "input_data": {"Market Value": "12000000", ...}                  │   │
│  │   },                                                                 │   │
│  │   {                                                                  │   │
│  │     "index": 1,                                                      │   │
│  │     "section_name": "top_movers",                                    │   │
│  │     "asset_id": "a9fdeb96-cdfa-4a48-9467-bf97a06f8738",              │   │
│  │     "input_data": {"top_movers_data": "..."}                         │   │
│  │   },                                                                 │   │
│  │   {                                                                  │   │
│  │     "index": 2,                                                      │   │
│  │     "section_name": "model_and_rebalancing",                         │   │
│  │     "asset_id": "0edec5d8-a1db-44c7-bdf5-3b028be0efc6",              │   │
│  │     "input_data": {"total_portfolio_mv": 1000000, ...}               │   │
│  │   }                                                                  │   │
│  │ ]                                                                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  S3 UPDATE:                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ {                                                                    │   │
│  │   "status": "detecting_section",                                     │   │
│  │   "progress": 40,                                                    │   │
│  │   "section_detected": "portfolio_snapshot, top_movers, model_and_...",│  │
│  │   "trace": [..., {"step": "2. Detect Sections", "status": "completed"}]│ │
│  │ }                                                                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### **STEP 5: Parallel Section Processing**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 5: PARALLEL SECTION PROCESSING                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  INPUT (3 sections to process in parallel):                                 │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ Section 1: portfolio_snapshot                                        │   │
│  │ Section 2: top_movers                                                │   │
│  │ Section 3: model_and_rebalancing                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  PROCESSING (asyncio.gather - runs in parallel):                            │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                                                                      │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │   │
│  │  │  Section 1   │  │  Section 2   │  │  Section 3   │               │   │
│  │  │              │  │              │  │              │               │   │
│  │  │ 1.Create Conv│  │ 1.Create Conv│  │ 1.Create Conv│               │   │
│  │  │ 2.Send Prompt│  │ 2.Send Prompt│  │ 2.Send Prompt│               │   │
│  │  │ 3.Poll Resp  │  │ 3.Poll Resp  │  │ 3.Poll Resp  │               │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘               │   │
│  │         │                 │                 │                        │   │
│  │         ▼                 ▼                 ▼                        │   │
│  │  ┌──────────────────────────────────────────────────────┐           │   │
│  │  │              asyncio.gather()                         │           │   │
│  │  │  Waits for ALL sections to complete                   │           │   │
│  │  └──────────────────────────────────────────────────────┘           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### **STEP 5A: Single Section Processing (Detail)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 5A: SINGLE SECTION PROCESSING (e.g., portfolio_snapshot)               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  SUB-STEP 5A.1: CREATE CONVERSATION                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ INPUT:                                                               │   │
│  │   asset_id: "c7b75809-063a-4f9c-80a8-496e88ed4894"                   │   │
│  │   access_token: "eyJhbGciOiJSUzI1NiIsInR5cCI..."                     │   │
│  │                                                                      │   │
│  │ API CALL:                                                            │   │
│  │   POST /magicplatform/v1/genai/conversation/create                   │   │
│  │   Body: {"conversation_name": "AI Insights", "asset_version_id": ...}│   │
│  │                                                                      │   │
│  │ OUTPUT:                                                              │   │
│  │   conv_id: "6953cc347432dcd48967cb34"                                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  SUB-STEP 5A.2: SEND PROMPT                                                 │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ INPUT:                                                               │   │
│  │   conv_id: "6953cc347432dcd48967cb34"                                │   │
│  │   prompt: '{"Market Value": "12000000", "Investment Value": ...}'    │   │
│  │                                                                      │   │
│  │ API CALL:                                                            │   │
│  │   POST /magicplatform/v1/genai/conversation/addmessage/stream        │   │
│  │                                                                      │   │
│  │ OUTPUT:                                                              │   │
│  │   message_id: "6953cc34dd24cce8ffb41f33"                             │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  SUB-STEP 5A.3: POLL FOR RESPONSE                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ INPUT:                                                               │   │
│  │   conv_id: "6953cc347432dcd48967cb34"                                │   │
│  │   message_id: "6953cc34dd24cce8ffb41f33"                             │   │
│  │                                                                      │   │
│  │ API CALL (repeated until SUCCESS):                                   │   │
│  │   GET /magicplatform/v1/genai/conversation/response/{conv}/{msg}     │   │
│  │                                                                      │   │
│  │ POLLING LOOP:                                                        │   │
│  │   Attempt 1: status=IN_PROGRESS → await asyncio.sleep(2)             │   │
│  │   Attempt 2: status=IN_PROGRESS → await asyncio.sleep(2)             │   │
│  │   ...                                                                │   │
│  │   Attempt N: status=SUCCESS → return response                        │   │
│  │                                                                      │   │
│  │ OUTPUT:                                                              │   │
│  │   response: '{"UGL": "+", "XIRR_SI vs BM": "positive", ...}'         │   │
│  │   cost: 0.08013224999999999                                          │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  SECTION RESULT:                                                            │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ {                                                                    │   │
│  │   "section": "portfolio_snapshot",                                   │   │
│  │   "status": "completed",                                             │   │
│  │   "conv_id": "6953cc347432dcd48967cb34",                             │   │
│  │   "message_id": "6953cc34dd24cce8ffb41f33",                          │   │
│  │   "response": '{"UGL": "+", "XIRR_SI vs BM": "positive", ...}',      │   │
│  │   "cost": 0.08013224999999999,                                       │   │
│  │   "error": null                                                      │   │
│  │ }                                                                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### **STEP 6: Aggregate Results & Complete**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 6: AGGREGATE RESULTS & COMPLETE                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  INPUT (results from all 3 parallel sections):                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ results = [                                                          │   │
│  │   {section: "portfolio_snapshot", status: "completed", cost: 0.08},  │   │
│  │   {section: "top_movers", status: "completed", cost: 0.24},          │   │
│  │   {section: "model_and_rebalancing", status: "completed", cost: 0.004}│  │
│  │ ]                                                                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  PROCESSING:                                                                │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 1. Aggregate all conv_ids: "conv1, conv2, conv3"                     │   │
│  │ 2. Aggregate all message_ids: "msg1, msg2, msg3"                     │   │
│  │ 3. Sum total cost: 0.08 + 0.24 + 0.004 = 0.324                       │   │
│  │ 4. Check if all completed successfully                               │   │
│  │ 5. Build result_data dictionary                                      │   │
│  │ 6. Set final status (COMPLETED or ERROR)                             │   │
│  │ 7. Save final state to S3                                            │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  FINAL S3 OUTPUT:                                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ s3://wealth-ai-tasks/tasks/task_20251230182721_01e09305.json         │   │
│  │ {                                                                    │   │
│  │   "task_id": "task_20251230182721_01e09305",                         │   │
│  │   "status": "completed",                                             │   │
│  │   "message": "All 3 section(s) completed successfully",              │   │
│  │   "progress": 100,                                                   │   │
│  │   "section_detected": "portfolio_snapshot, top_movers, model_...",   │   │
│  │   "conv_id": "conv1, conv2, conv3",                                  │   │
│  │   "message_id": "msg1, msg2, msg3",                                  │   │
│  │   "result_data": {                                                   │   │
│  │     "portfolio_snapshot": {...},                                     │   │
│  │     "top_movers": {...},                                             │   │
│  │     "model_and_rebalancing": {...}                                   │   │
│  │   },                                                                 │   │
│  │   "cost": 0.324,                                                     │   │
│  │   "completed_at": "2025-12-30T18:28:08.881253",                       │   │
│  │   "trace": [... all trace entries ...]                               │   │
│  │ }                                                                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### **STEP 7: User Retrieves Result**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 7: USER RETRIEVES RESULT                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  INPUT (User Request):                                                      │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ GET /task/task_20251230182721_01e09305/result                        │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  PROCESSING:                                                                │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ 1. Check in-memory cache first                                       │   │
│  │ 2. If not in cache, fetch from S3 (async)                            │   │
│  │ 3. Calculate elapsed_seconds                                         │   │
│  │ 4. Return TaskResultResponse                                         │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  OUTPUT (to User):                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ {                                                                    │   │
│  │   "task_id": "task_20251230182721_01e09305",                         │   │
│  │   "status": "completed",                                             │   │
│  │   "section_detected": "portfolio_snapshot, top_movers, model_...",   │   │
│  │   "conv_id": "conv1, conv2, conv3",                                  │   │
│  │   "message_id": "msg1, msg2, msg3",                                  │   │
│  │   "data": {                                                          │   │
│  │     "portfolio_snapshot": {                                          │   │
│  │       "status": "completed",                                         │   │
│  │       "response": "...",                                             │   │
│  │       "cost": 0.08                                                   │   │
│  │     },                                                               │   │
│  │     "top_movers": {...},                                             │   │
│  │     "model_and_rebalancing": {...}                                   │   │
│  │   },                                                                 │   │
│  │   "cost": 0.324,                                                     │   │
│  │   "elapsed_seconds": 47.05                                           │   │
│  │ }                                                                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## S3 Storage Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ S3 BUCKET: wealth-ai-tasks                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ tasks/                                                                      │
│ ├── task_20251230160634_60766c26.json    ← Individual file per task         │
│ ├── task_20251230160635_950ca81a.json                                       │
│ ├── task_20251230163956_893eaadb.json                                       │
│ ├── task_20251230164000_adcd912a.json                                       │
│ ├── task_20251230164123_56b3adf2.json                                       │
│ ├── task_20251230165920_10a9b077.json                                       │
│ ├── task_20251230165928_141c2938.json                                       │
│ ├── task_20251230174735_eb546eaa.json                                       │
│ ├── task_20251230180134_3ba97c86.json                                       │
│ ├── task_20251230182721_01e09305.json                                       │
│ └── task_20251230212904_4cfc1734.json                                       │
│                                                                             │
│ Benefits:                                                                   │
│ ├── No race conditions (each file is independent)                           │
│ ├── Multiple pods can write simultaneously                                  │
│ ├── O(1) access to individual tasks                                         │
│ └── Auto-cleanup via S3 lifecycle policy (30 days)                          │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Component Interaction Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        COMPONENT INTERACTION                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐                                                            │
│  │   User      │                                                            │
│  └──────┬──────┘                                                            │
│         │ HTTP Request                                                      │
│         ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         FastAPI (api_task_1.py)                      │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐             │   │
│  │  │ /submit  │  │ /status  │  │ /result  │  │ /tasks   │             │   │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘             │   │
│  └───────┼─────────────┼─────────────┼─────────────┼────────────────────┘   │
│          │             │             │             │                        │
│          ▼             ▼             ▼             ▼                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      TaskManager (task_manager.py)                   │   │
│  │                                                                      │   │
│  │  ┌─────────────┐  ┌─────────────────┐  ┌───────────────────────┐    │   │
│  │  │ submit_task │  │ Background      │  │ In-Memory Cache       │    │   │
│  │  │             │  │ Workers (async) │  │ Dict[str, Task]       │    │   │
│  │  └──────┬──────┘  └────────┬────────┘  └───────────┬───────────┘    │   │
│  │         │                  │                       │                 │   │
│  │         │    ┌─────────────┴─────────────┐        │                 │   │
│  │         │    │     asyncio.Queue         │        │                 │   │
│  │         │    │   [task_id1, task_id2...] │        │                 │   │
│  │         │    └───────────────────────────┘        │                 │   │
│  └─────────┼─────────────────────────────────────────┼──────────────────┘   │
│            │                                         │                      │
│            ▼                                         ▼                      │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │              S3AsyncIndividualStorage (task_storage_s3_async.py)     │   │
│  │                                                                      │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │   │
│  │  │ save_task() │  │ get_task()  │  │ delete_task()│                  │   │
│  │  │   (async)   │  │   (async)   │  │   (async)   │                  │   │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                  │   │
│  └─────────┼────────────────┼────────────────┼──────────────────────────┘   │
│            │                │                │                              │
│            └────────────────┼────────────────┘                              │
│                             │                                               │
│                             ▼                                               │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         AWS S3 (aioboto3)                            │   │
│  │                                                                      │   │
│  │  s3://wealth-ai-tasks/tasks/                                         │   │
│  │  ├── task_001.json                                                   │   │
│  │  ├── task_002.json                                                   │   │
│  │  └── ...                                                             │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Files to Create/Modify

| File | Action | Description |
|------|--------|-------------|
| `task_storage_s3_async_individual.py` | **CREATE** | New S3 async storage class |
| `task_manager.py` | **MODIFY** | Use S3 storage instead of local JSON |
| `api_task_1.py` | **MODIFY** | Minor updates for async operations |
| `requirements.txt` | **MODIFY** | Add `aioboto3`, `aiohttp` |
| `.env` | **MODIFY** | Add AWS credentials |

---
