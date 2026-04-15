# ARCHITECTURE.md
# Internal Photo Object Detection Platform

_Last updated: 2026-04-15_

---

## 1. System Overview

This is an internal web application that allows a small team of 5 users to upload photos into
named projects and run asynchronous custom object detection jobs against those photos. Detection
results — bounding boxes and confidence scores — are stored and displayed in a review UI.
Authentication is enforced via Azure Entra ID SSO (MSAL); there is no public access.
The system is built entirely on managed/serverless Azure services to minimise operational overhead.

---

## 2. Component Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Azure Entra ID (SSO)                               │
│                    Tenant-scoped App Registration                           │
└───────────────────────────┬─────────────────────────────────────────────────┘
                            │  MSAL token (OIDC/OAuth2)
                            ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Azure Static Web Apps                                     │
│               React SPA  (built-in Entra ID auth route)                     │
│                                                                             │
│   ┌──────────────┐  ┌──────────────┐  ┌───────────────┐  ┌──────────────┐ │
│   │  Projects UI │  │  Upload UI   │  │  Jobs Status  │  │  Results UI  │ │
│   │  (CRUD)      │  │  (drag/drop) │  │  (poll/badge) │  │  (bbox view) │ │
│   └──────────────┘  └──────────────┘  └───────────────┘  └──────────────┘ │
└───────────────────────────┬─────────────────────────────────────────────────┘
                            │  HTTPS + Bearer JWT (Entra ID token)
                            ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Azure Container Apps  (Backend API)                       │
│                    Node.js / FastAPI  — REST API                            │
│                                                                             │
│   ┌──────────────────────────────────────────────────────────────────────┐ │
│   │  Auth middleware → validates Entra ID JWT on every request           │ │
│   │  Projects handler │ Photos handler │ Jobs handler │ Results handler  │ │
│   └──────────────────────────────────────────────────────────────────────┘ │
│                    │                      │                                  │
│         ┌──────────┘                      └───────────┐                     │
│         ▼                                             ▼                     │
│  ┌──────────────────┐                  ┌─────────────────────────────────┐  │
│  │  Azure Cosmos DB │                  │       Azure Blob Storage        │  │
│  │  (serverless)    │                  │  Container: photos-raw          │  │
│  │                  │                  │  Container: photos-processed    │  │
│  │  - users         │                  │  (private, SAS URLs for access) │  │
│  │  - projects      │                  └──────────────┬──────────────────┘  │
│  │  - photos        │                                 │                     │
│  │  - detection_jobs│                                 │  Blob trigger /     │
│  │  - det_results   │                                 │  Queue message      │
│  └──────────────────┘                                 │                     │
│                                                       ▼                     │
│                                        ┌──────────────────────────────────┐ │
│                                        │   Azure Storage Queue            │ │
│                                        │   detection-jobs-queue           │ │
│                                        └──────────────┬───────────────────┘ │
└──────────────────────────────────────────────────────┼─────────────────────┘
                                                        │  Queue trigger
                                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                Azure Container Apps Job  (Detection Worker)                  │
│                Python  —  queue-triggered, runs to completion               │
│                                                                             │
│   1. Dequeue job message (jobId, photoId, blobPath)                        │
│   2. Download photo from Blob Storage                                       │
│   3. Call Azure AI Custom Vision Prediction API                             │
│   4. Parse bounding boxes + confidence scores                               │
│   5. Write DetectionResult records to Cosmos DB                             │
│   6. Update DetectionJob status → "completed" | "failed"                   │
└──────────────────────────────┬──────────────────────────────────────────────┘
                               │  HTTP Prediction API
                               ▼
              ┌─────────────────────────────────┐
              │   Azure AI Custom Vision        │
              │   (Prediction endpoint)         │
              │   Custom-trained object model   │
              └─────────────────────────────────┘
```

---

## 3. Tech Stack

| Layer | Choice | Justification |
|---|---|---|
| **Frontend SPA** | React 18 + TypeScript, hosted on **Azure Static Web Apps** | Built-in Entra ID auth integration (no custom auth server needed), global CDN, free tier sufficient for 5 users, staging environments included |
| **Backend API** | **Azure Container Apps** running Node.js (Fastify) or Python (FastAPI) | Serverless-scale-to-zero when idle, supports long-running HTTP requests, easier to containerise custom logic vs. Azure Functions cold-start limits; managed ingress/TLS |
| **Auth** | **Azure Entra ID** + **MSAL.js** (frontend) + JWT validation middleware (backend) | Requirement mandates Entra ID SSO; MSAL handles token acquisition, refresh, and silent re-auth; backend validates `aud`/`iss`/`exp` claims on every request |
| **File Storage** | **Azure Blob Storage** (two containers: `photos-raw`, `photos-processed`) | Native Azure integration, per-object SAS URLs for time-limited secure download, lifecycle policies for cost control, scalable well beyond 300 photos |
| **Database** | **Azure Cosmos DB for NoSQL** (serverless capacity mode) | Serverless billing fits the low-throughput profile (5 users, ~300 photos); flexible JSON document model suits detection results (variable bounding box arrays); built-in partitioning; no server to manage |
| **Async Job Queue** | **Azure Storage Queue** | Sufficient for this scale (no ordering guarantees needed, no dead-letter retry complexity of Service Bus required at this volume); integrates natively with Container Apps Jobs triggers; near-zero cost |
| **Detection Worker** | **Azure Container Apps Job** (queue-triggered, consumption profile) | Runs to completion on queue message arrival, scales to 0 when idle, full container flexibility for Python ML dependencies, no timeout ceiling unlike Azure Functions (10-min limit) |
| **Object Detection** | **Azure AI Custom Vision** (Prediction API) | Fully managed training and inference; no GPU infrastructure to operate; REST prediction endpoint; custom model trained on domain-specific objects; fits the internal/low-volume use case |
| **Hosting / Compute** | **Azure Container Apps Environment** (shared) for API + Worker | Single managed environment handles networking, TLS, scaling, and secret injection via managed identity; no AKS cluster overhead |
| **Secrets Management** | **Azure Key Vault** + Managed Identity | API and Worker access Cosmos DB connection string, Blob Storage key, and Custom Vision key via managed identity — no secrets in environment variables or code |
| **Monitoring** | **Azure Monitor + Application Insights** | Distributed tracing across API and Worker, structured logs, alerting on job failures; built into Container Apps with one flag |

---

## 4. Data Model

All entities are stored in **Azure Cosmos DB for NoSQL**, partitioned as noted.
The document model is preferred here because `DetectionResult` contains a variable-length array
of bounding boxes that maps cleanly to JSON.

### 4.1 User

Represents an authenticated internal user. Populated on first login from Entra ID token claims.
No passwords stored — identity is delegated entirely to Entra ID.

```
User {
  id:           string        // Entra ID object ID (oid claim) — partition key
  email:        string        // upn or email claim
  displayName:  string        // name claim
  createdAt:    ISO8601
  lastLoginAt:  ISO8601
}
```

Partition key: `/id`

### 4.2 Project

A named workspace that groups photos and their detection jobs.

```
Project {
  id:          string (ULID)   // partition key
  name:        string
  description: string?
  ownerId:     string          // User.id
  memberIds:   string[]        // User.id list (for shared access)
  createdAt:   ISO8601
  updatedAt:   ISO8601
  status:      "active" | "archived"
}
```

Partition key: `/id`

### 4.3 Photo

Metadata record for an uploaded image. The actual binary is in Blob Storage.

```
Photo {
  id:            string (ULID)   // partition key
  projectId:     string          // Project.id  — secondary index
  uploadedBy:    string          // User.id
  filename:      string          // original filename
  blobPath:      string          // Blob Storage path: photos-raw/{projectId}/{photoId}/{filename}
  contentType:   string          // e.g. "image/jpeg"
  sizeBytes:     number
  width:         number?         // pixels, populated post-upload
  height:        number?
  uploadedAt:    ISO8601
  tags:          string[]?       // optional user-defined tags
}
```

Partition key: `/id`
Secondary index on `projectId` for listing photos by project.

### 4.4 DetectionJob

Tracks one asynchronous detection run against a single photo.

```
DetectionJob {
  id:             string (ULID)   // partition key
  photoId:        string          // Photo.id
  projectId:      string          // denormalised for query convenience
  triggeredBy:    string          // User.id
  modelVersion:   string          // Custom Vision iteration name/id
  status:         "queued" | "processing" | "completed" | "failed"
  queuedAt:       ISO8601
  startedAt:      ISO8601?
  completedAt:    ISO8601?
  errorMessage:   string?
  durationMs:     number?
}
```

Partition key: `/id`
Secondary index on `photoId` and `projectId`.

### 4.5 DetectionResult

Stores the output of a completed detection job — one document per job.

```
DetectionResult {
  id:           string          // == DetectionJob.id (1:1 relationship)
  jobId:        string          // DetectionJob.id — partition key
  photoId:      string
  projectId:    string
  modelVersion: string
  createdAt:    ISO8601
  predictions:  Prediction[]
}

Prediction {
  tagName:      string          // object class label, e.g. "crack", "bolt"
  probability:  float           // confidence score 0.0 – 1.0
  boundingBox: {
    left:       float           // normalised 0.0–1.0 (fraction of image width)
    top:        float
    width:      float
    height:     float
  }
}
```

Partition key: `/jobId`

### Entity Relationship Summary

```
User ──< Project (owner)
User ──< Project (member, many-to-many via memberIds[])
Project ──< Photo
Photo ──< DetectionJob
DetectionJob ──1 DetectionResult
```

---

## 5. API Design

Base URL: `https://api.<env>.internal.example.com/v1`

All endpoints require `Authorization: Bearer <Entra ID access token>`.
The backend validates token signature, `aud` (app client ID), `iss`, and `exp` on every request.
HTTP `401` is returned for missing/invalid tokens; `403` for valid token but insufficient access.

### 5.1 Auth

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/auth/me` | Returns current user profile (decoded from JWT + DB record). Used by SPA on load to confirm session and retrieve `displayName`. |

Response `200`:
```json
{
  "id": "aad-oid-abc123",
  "email": "alice@company.com",
  "displayName": "Alice Smith",
  "lastLoginAt": "2026-04-15T10:00:00Z"
}
```

---

### 5.2 Projects

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/projects` | List all projects the caller owns or is a member of. |
| `POST` | `/projects` | Create a new project. |
| `GET` | `/projects/:projectId` | Get a single project by ID. |
| `PATCH` | `/projects/:projectId` | Update name, description, or memberIds. |
| `DELETE` | `/projects/:projectId` | Soft-delete (status → archived). Owner only. |

`POST /projects` request body:
```json
{
  "name": "Bridge Inspection Q1 2026",
  "description": "Quarterly corrosion survey photos"
}
```

`GET /projects` response `200`:
```json
{
  "items": [
    {
      "id": "01HXZ...",
      "name": "Bridge Inspection Q1 2026",
      "ownerId": "aad-oid-abc123",
      "memberIds": ["aad-oid-def456"],
      "createdAt": "2026-04-01T08:00:00Z",
      "status": "active",
      "photoCount": 42
    }
  ]
}
```

---

### 5.3 Photos

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/projects/:projectId/photos` | List all photos in a project. |
| `POST` | `/projects/:projectId/photos/upload-url` | Request a short-lived SAS upload URL for direct browser-to-Blob upload (avoids proxying binary through the API). Returns `uploadUrl` + `photoId`. |
| `POST` | `/projects/:projectId/photos/:photoId/confirm` | Called by SPA after successful direct Blob upload to confirm and persist Photo metadata. |
| `GET` | `/projects/:projectId/photos/:photoId` | Get photo metadata + short-lived SAS download URL. |
| `DELETE` | `/projects/:projectId/photos/:photoId` | Delete photo and associated jobs/results. |

`POST /projects/:projectId/photos/upload-url` response `200`:
```json
{
  "photoId": "01HYA...",
  "uploadUrl": "https://mystorageaccount.blob.core.windows.net/photos-raw/...?sv=...&se=...&sig=...",
  "expiresAt": "2026-04-15T10:15:00Z"
}
```

`POST /projects/:projectId/photos/:photoId/confirm` request body:
```json
{
  "filename": "bridge-north-face.jpg",
  "contentType": "image/jpeg",
  "sizeBytes": 2048576
}
```

---

### 5.4 Detection Jobs

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/projects/:projectId/photos/:photoId/jobs` | Trigger a new detection job. Creates `DetectionJob` (status: `queued`), enqueues message to Storage Queue. |
| `GET` | `/projects/:projectId/photos/:photoId/jobs` | List all jobs for a photo (latest first). |
| `GET` | `/projects/:projectId/photos/:photoId/jobs/:jobId` | Poll job status. SPA calls this every 5s until `completed` or `failed`. |

`POST .../jobs` response `202 Accepted`:
```json
{
  "jobId": "01HYB...",
  "status": "queued",
  "queuedAt": "2026-04-15T10:01:00Z"
}
```

`GET .../jobs/:jobId` response `200`:
```json
{
  "jobId": "01HYB...",
  "status": "completed",
  "queuedAt": "2026-04-15T10:01:00Z",
  "startedAt": "2026-04-15T10:01:03Z",
  "completedAt": "2026-04-15T10:01:09Z",
  "durationMs": 6210,
  "modelVersion": "Iteration3"
}
```

---

### 5.5 Detection Results

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/projects/:projectId/photos/:photoId/jobs/:jobId/results` | Retrieve bounding boxes and confidence scores for a completed job. Returns `404` if job is not yet `completed`. |

Response `200`:
```json
{
  "jobId": "01HYB...",
  "photoId": "01HYA...",
  "modelVersion": "Iteration3",
  "photoDownloadUrl": "https://...blob...?sas...",
  "predictions": [
    {
      "tagName": "corrosion",
      "probability": 0.94,
      "boundingBox": {
        "left": 0.12,
        "top": 0.34,
        "width": 0.08,
        "height": 0.06
      }
    },
    {
      "tagName": "crack",
      "probability": 0.71,
      "boundingBox": {
        "left": 0.55,
        "top": 0.20,
        "width": 0.12,
        "height": 0.04
      }
    }
  ]
}
```

The SPA renders this response by overlaying `<canvas>` bounding boxes on the photo image, using
the normalised coordinates (`left * imageWidth`, `top * imageHeight`, etc.).

---

### 5.6 Error Response Schema

All errors follow a consistent envelope:

```json
{
  "error": {
    "code": "PHOTO_NOT_FOUND",
    "message": "Photo 01HYA... does not exist in project 01HXZ...",
    "requestId": "req-abc-123"
  }
}
```

Standard HTTP status codes: `400` bad request, `401` unauthenticated, `403` forbidden,
`404` not found, `409` conflict, `422` validation error, `500` internal error.

---

## 6. Deployment Architecture

### 6.1 Azure Resource Map

```
Azure Subscription
└── Resource Group: rg-photodetect-prod
    │
    ├── Azure Entra ID
    │   └── App Registration: photodetect-app
    │       ├── Redirect URI: https://<app>.azurestaticapps.net/.auth/login/aad/callback
    │       └── Expose API scope: api://photodetect/access_as_user
    │
    ├── Azure Static Web Apps: stapp-photodetect-prod
    │   ├── Framework: React (Vite build)
    │   ├── Auth provider: Azure Active Directory (built-in)
    │   ├── Routes: /*.html → SPA, /api/* → linked backend (or direct Container Apps URL)
    │   └── Environments: production, staging (PR previews)
    │
    ├── Azure Container Apps Environment: cae-photodetect-prod
    │   ├── Container App: ca-api-photodetect
    │   │   ├── Image: ghcr.io/org/photodetect-api:latest
    │   │   ├── Min replicas: 0  (scale to zero when no traffic)
    │   │   ├── Max replicas: 3
    │   │   ├── Ingress: external HTTPS, port 8080
    │   │   ├── Secrets: via Key Vault references + managed identity
    │   │   └── Health probe: GET /health → 200
    │   │
    │   └── Container Apps Job: caj-worker-photodetect
    │       ├── Image: ghcr.io/org/photodetect-worker:latest
    │       ├── Trigger: Azure Storage Queue (detection-jobs-queue)
    │       ├── Polling interval: 30s
    │       ├── Parallelism: 1 (sufficient for 5 users)
    │       └── Secrets: via Key Vault references + managed identity
    │
    ├── Azure Cosmos DB Account: cosmos-photodetect-prod
    │   ├── Capacity: Serverless
    │   ├── Database: photodetect
    │   └── Containers (with partition keys):
    │       ├── users         /id
    │       ├── projects      /id
    │       ├── photos        /id
    │       ├── detection_jobs /id
    │       └── detection_results /jobId
    │
    ├── Azure Storage Account: stphotodetectprod
    │   ├── Blob containers:
    │   │   ├── photos-raw          (private)
    │   │   └── photos-processed    (private)
    │   └── Queue: detection-jobs-queue
    │
    ├── Azure AI Custom Vision: cogcv-photodetect-prod
    │   ├── Training resource  (used offline to train/publish iterations)
    │   └── Prediction resource (called by Worker at runtime)
    │
    ├── Azure Key Vault: kv-photodetect-prod
    │   ├── Secret: cosmos-connection-string
    │   ├── Secret: storage-connection-string
    │   └── Secret: customvision-prediction-key
    │
    └── Azure Application Insights: appi-photodetect-prod
        └── Connected to Container Apps Environment (auto-instrumentation)
```

### 6.2 Managed Identity Chain

```
ca-api-photodetect   →  System-assigned Managed Identity
  → Key Vault Secrets User role on kv-photodetect-prod
  → Storage Blob Data Contributor on stphotodetectprod
  → Cosmos DB Built-in Data Contributor on cosmos-photodetect-prod

caj-worker-photodetect  →  System-assigned Managed Identity
  → Key Vault Secrets User role on kv-photodetect-prod
  → Storage Blob Data Reader on stphotodetectprod
  → Storage Queue Data Message Processor on stphotodetectprod
  → Cosmos DB Built-in Data Contributor on cosmos-photodetect-prod
```

No connection strings or keys are stored in container environment variables. All secrets are
resolved at startup via Key Vault SDK using the managed identity credential.

### 6.3 CI/CD Pipeline (GitHub Actions)

```
on: push to main
  ├── build-api job
  │   ├── docker build photodetect-api
  │   ├── push to GitHub Container Registry
  │   └── az containerapp update --image ...
  │
  ├── build-worker job
  │   ├── docker build photodetect-worker
  │   ├── push to GitHub Container Registry
  │   └── az containerapp job update --image ...
  │
  └── build-frontend job
      ├── npm run build
      └── az staticwebapp deploy ...

on: pull_request
  └── Static Web Apps auto-creates staging preview environment
```

### 6.4 Async Job Flow (End-to-End)

```
1.  User clicks "Run Detection" in SPA
2.  SPA calls POST /projects/:pid/photos/:photoId/jobs
3.  API creates DetectionJob (status: "queued") in Cosmos DB
4.  API enqueues message to Azure Storage Queue:
    { "jobId": "...", "photoId": "...", "blobPath": "photos-raw/..." }
5.  API returns 202 to SPA with jobId
6.  SPA begins polling GET .../jobs/:jobId every 5 seconds
7.  Container Apps Job detects queue message (30s poll interval)
8.  Worker sets job status → "processing" in Cosmos DB
9.  Worker downloads photo from Blob Storage using managed identity
10. Worker calls Custom Vision Prediction API (REST)
11. Worker parses response: tags, probabilities, bounding boxes
12. Worker writes DetectionResult document to Cosmos DB
13. Worker sets job status → "completed" in Cosmos DB
14. Worker deletes queue message (acknowledges completion)
15. SPA poll detects status "completed"
16. SPA calls GET .../jobs/:jobId/results
17. SPA renders bounding boxes on photo canvas
```

---

## 7. Security

- **Authentication**: All routes (API + Static Web App) require a valid Azure Entra ID JWT.
  The SPA uses `@azure/msal-react` for silent token acquisition and redirect flows.
  The API validates token on every request: signature (JWKS), `aud`, `iss`, `exp`, `nbf`.
- **Authorisation**: Project-scoped access checks — users can only access projects they own or
  are members of. `ownerId` check is enforced server-side; the frontend is not trusted for authz.
- **No public endpoints**: Azure Static Web App has built-in auth gate (all routes protected).
  Container Apps ingress is HTTPS-only; HTTP is rejected. No blob container is set to public.
- **SAS URLs**: Photos are served via short-lived SAS URLs (15-minute expiry) generated
  server-side. Browsers never receive a long-lived credential.
- **Secrets**: All secrets (Cosmos DB key, Custom Vision key, Storage key) live in Key Vault and
  are accessed via managed identity — no secrets in code, env vars, or container images.
- **Input validation**: API validates all request bodies with a schema library (Zod / Pydantic).
  File uploads are validated for MIME type and size (max 20MB) before SAS URL is issued.
- **SSRF risk**: The Worker calls the Custom Vision prediction endpoint (a fixed, well-known Azure
  URL). No user-controlled URLs are fetched. The blob path for photo download is constructed
  server-side from database values, not from user input.
- **Encryption**: All data in transit uses TLS 1.2+. Cosmos DB and Blob Storage encrypt at rest
  by default (AES-256, Microsoft-managed keys). Key Vault secrets are encrypted at rest.

---

## 8. Operability

- **Logging**: Structured JSON logs (correlation ID on every request) emitted to stdout by
  the API and Worker; Container Apps forwards to Application Insights automatically.
- **Distributed tracing**: Application Insights traces span API request → queue enqueue →
  Worker dequeue → Custom Vision call → Cosmos DB write. The `jobId` is used as the
  correlation key across all components.
- **Alerting**: Azure Monitor alert rules on:
  - `DetectionJob` failure rate > 10% in 1h
  - API 5xx rate > 5% in 5min
  - Storage Queue depth > 20 messages (worker falling behind)
- **Health check**: `GET /health` on the API returns `200 { "status": "ok" }`. Container Apps
  uses this as the liveness probe.
- **Deployment**: Zero-downtime rolling update via Container Apps revision management.
  Static Web Apps staging environment allows PR-level preview before production merge.
- **Rollback**: Container Apps supports instant revision rollback via `az containerapp revision
  set-mode` — traffic can be shifted back to the previous revision in under 30 seconds.

---

## 9. Risks & Open Questions

### Technical Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|------|-----------|--------|------------|
| 1 | **Custom Vision model accuracy is poor for the domain** | Low | High | Model is already trained and available. If accuracy proves insufficient in production, the Worker's `ObjectDetectionService` interface can be swapped for a container-hosted ONNX/YOLO model without changes to the API or frontend. |
| 2 | **Container Apps Job cold-start adds latency** (queue poll interval 30s) | Medium | Low | For 5 users this is acceptable. If sub-10s response is required, switch to Azure Functions with Storage Queue trigger (sub-second cold start) or keep a minimum 1 replica running. |
| 3 | **Cosmos DB serverless RU bursting costs** | Low | Low | At ~300 photos and 5 users, consumption is negligible. Set a budget alert at $50/month. If usage grows, evaluate provisioned throughput. |
| 4 | **Custom Vision retirement or pricing change** | Low | Medium | The Worker is isolated behind a single `ObjectDetectionService` interface; swapping to a container-hosted ONNX model requires only changes to the Worker, not the API or frontend. |
| 5 | **Blob SAS URL leakage** | Low | Medium | 15-minute SAS expiry limits exposure window. Access logging on the Storage Account will capture all SAS URL accesses for audit. |

### Open Questions

1. ~~**What objects are being detected?**~~ **Resolved.** A trained Custom Vision model is already
   available and published. The Worker can reference the existing prediction endpoint and
   iteration name directly. No training effort is on the critical path.

2. ~~**Should multiple detection models/iterations be selectable per job?**~~ **Assumed: No.**
   The Worker always calls the single latest published Custom Vision iteration. `modelVersion`
   is recorded on each job/result for auditability, but users cannot select a version at trigger
   time. Can be added later if needed.

3. ~~**Is there a need for batch triggering?**~~ **Assumed: Yes — add a batch endpoint.**
   A `POST /projects/:projectId/jobs/batch` endpoint enqueues a detection job for every photo
   in the project that does not already have a `completed` job. This is the most common workflow
   (upload a set of photos, run detection on all of them). Single-photo triggering is also kept.

4. ~~**Notification vs. polling?**~~ **Assumed: Polling only.**
   5-second polling is sufficient for 5 users. Azure Web PubSub is not added; it would
   introduce an extra managed service for negligible UX gain at this scale.

5. ~~**Data retention policy?**~~ **Assumed: Indefinite retention.**
   No lifecycle policies or TTLs are configured. At ~300 photos the storage cost is negligible
   (< $1/month). This can be revisited if the photo corpus grows significantly.

6. ~~**Photo deduplication?**~~ **Assumed: No deduplication.**
   Duplicate uploads create separate Photo records. At 300 photos and 5 users, accidental
   duplicates are rare and the overhead of content-hash checking is not justified.

---

## 10. Cost Estimate Summary

_Detailed breakdown: see `/COST-ESTIMATE.md`_

### Assumptions
- 5 internal users, ~300 photos (~900 MB), ~50 detection jobs/month, ~15,000 API requests/month
- Azure Consumption / Serverless pricing, East US region

### Monthly Estimates

| Component | Low/mo | Mid/mo | High/mo |
|---|---|---|---|
| Azure Static Web Apps | $0.00 | $0.00 | $9.00 |
| Container Apps — Backend API | $0.50 | $2.00 | $12.00 |
| Container Apps Job — Detection Worker | $0.10 | $0.40 | $4.00 |
| Cosmos DB (Serverless) | $0.08 | $0.50 | $5.00 |
| Blob Storage | $0.05 | $0.20 | $3.00 |
| Storage Queue | $0.00 | $0.00 | $0.01 |
| Azure AI Custom Vision (Prediction) | $0.00 | $2.00 | $10.00 |
| Key Vault | $0.03 | $0.05 | $0.30 |
| Application Insights / Monitor | $0.00 | $0.28 | $3.00 |
| **Total** | **$0.76** | **$5.43** | **$46.31** |

**Low** = current state (5 users, ~50 jobs/mo). **Mid** = 2–3x growth. **High** = 10x+ growth.

Primary cost risks at scale: Application Insights data ingestion volume, Custom Vision
transaction growth from batch jobs, and Cosmos DB cross-partition query RU costs.

---

## Appendix: Naming Conventions

| Resource type | Pattern | Example |
|---|---|---|
| Resource Group | `rg-<app>-<env>` | `rg-photodetect-prod` |
| Container Apps Env | `cae-<app>-<env>` | `cae-photodetect-prod` |
| Container App | `ca-<role>-<app>` | `ca-api-photodetect` |
| Container Apps Job | `caj-<role>-<app>` | `caj-worker-photodetect` |
| Static Web App | `stapp-<app>-<env>` | `stapp-photodetect-prod` |
| Cosmos DB account | `cosmos-<app>-<env>` | `cosmos-photodetect-prod` |
| Storage account | `st<app><env>` (no hyphens) | `stphotodetectprod` |
| Key Vault | `kv-<app>-<env>` | `kv-photodetect-prod` |
| App Insights | `appi-<app>-<env>` | `appi-photodetect-prod` |
| Custom Vision | `cogcv-<app>-<env>` | `cogcv-photodetect-prod` |
