# Architecture

[← Back to README](../README.md)

Luma is a single-machine application. One FastAPI process serves the statically exported Next.js interface and every
`/api/v1` route from the same origin. Model inference, explanations, storage and audit logging all run locally.

## Components

```mermaid
flowchart TB
    subgraph FE["Frontend (Next.js · React · TypeScript)"]
        UI["Clinician workspace<br/>intake · result · evidence · review archive"]
    end

    subgraph BE["Backend (FastAPI · Python)"]
        AUTH["Auth and sessions<br/>Argon2id · CSRF · throttling · RBAC"]
        VAL["Validation<br/>DICOM / PNG / JPEG · adult & frontal checks · scrubbing"]
        PRED["Locked predictor<br/>ONNX Runtime · 3-model ensemble · calibration"]
        OCC["Occlusion engine<br/>6×6 grid + refinement"]
        JOBS["Synthesis jobs<br/>cancellable · owner-scoped"]
        STORE[("Encrypted store<br/>SQLite + AES-GCM")]
        AUDIT[("Audit chain<br/>HMAC-linked events")]
    end

    UI --> AUTH --> VAL --> PRED --> OCC
    AUTH --> STORE
    AUTH --> AUDIT
    AUTH --> JOBS
    JOBS -.->|"opt-in"| EXT["OpenRouter"]

    style EXT fill:#fff4e5,stroke:#c98a1b,color:#3a2a05
```

## Request flow for one analysis

```mermaid
sequenceDiagram
    autonumber
    actor R as Reviewer
    participant API as FastAPI
    participant V as Validator
    participant M as ONNX ensemble
    participant S as Encrypted store
    participant A as Audit chain

    R->>API: Upload image (+ optional de-identified context)
    API->>V: Validate format, projection, adult/frontal
    V-->>API: Scrubbed in-memory image (original never stored)
    API->>M: Verify SHA-256 artefacts, then run 3 models
    M-->>API: Averaged logits → calibrated score
    API->>M: Occlusion passes (48 masked inputs)
    API->>S: Save encrypted derivative and result
    API->>A: Append HMAC-linked event
    API-->>R: Score, agreement and region evidence
```

## Optional report synthesis (opt-in)

The synthesis runs only when the reviewer asks for it. The local result stays fully usable if it is unavailable.

| Pass | Receives | Never receives |
|---|---|---|
| 1. Vision review | Recorded projection and a re-encoded, metadata-free grayscale PNG preview | Original file, DICOM headers, model score, explanations, clinical context |
| 2. Clinical synthesis | Validated pass-1 JSON, bounded local evidence, coarse de-identified context | Any image bytes or original metadata |

Both passes must return strict JSON that matches a fixed schema. A malformed response gets one repair attempt, and the
job then fails closed. Deterministic safety checks run after generation and override the model, blocking autonomous
diagnosis, treatment orders and invented history. The API key is read only by the backend and never reaches the
browser, logs or exports.

## Delivery

- **CI:** GitHub Actions runs ruff lint, pytest, the TypeScript type check and the Next.js production build on pull
  requests and pushes to main.
- **Containers:** Dockerfiles exist for the web application. A cloud deployment was designed as infrastructure-as-code
  but retired in favour of the local-first design, for privacy reasons.
