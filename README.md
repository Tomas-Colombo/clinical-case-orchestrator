# 🏥 Intelligent Medical Case Management Platform

> AI-powered orchestration system for clinical follow-up of occupational health cases, built with n8n, PostgreSQL, and LLM agents.

---

## Overview

This project is an end-to-end automation platform designed to assist occupational health teams in the weekly follow-up of medical absence cases. It transforms unstructured clinical text from spreadsheets into structured, traceable records — and uses AI agents to automatically classify each case's evolution using a **traffic light system (green / yellow / red)**.

The system respects the doctor's existing workflow (Google Sheets as the primary interface) while adding intelligent automation and explainable AI classification underneath.

---

## Architecture

```
Google Sheets (Doctor Interface)
        │
        │  HTTP Webhook (Google Apps Script trigger)
        ▼
    n8n Orchestrator
        │
        ├── TransformCasoFromExcel   → Normalize & clean clinical text
        ├── Insertar/Actualizar Caso → UPSERT logic into PostgreSQL
        ├── Actualizar Casos         → Detect new observations via AI
        └── Predicción Semáforo      → Generate color + justification + confidence
                │
                ├── OpenAI / Gemini 2.5 Flash (LLM Agent)
                └── PostgreSQL on Aiven (structured persistence)
```

**Stack:** n8n · PostgreSQL · Google Sheets · Google Apps Script · OpenAI API · Gemini 2.5 Flash · Docker · Cloudflare Tunnel

---

## Key Features

- **Traffic Light Classification** — Each case is evaluated against its expected clinical evolution and assigned a color (🟢 green / 🟡 yellow / 🔴 red) with a confidence score and written justification
- **Unstructured Text Normalization** — LLM agent extracts only the *new* clinical observation from each update, filtering out repeated historical content and medical abbreviations
- **Full Traceability** — Every control, prediction, and correction is stored as an immutable historical record in PostgreSQL
- **Human-in-the-Loop Feedback** — Doctors can validate or correct AI predictions directly from Sheets; corrections are persisted to support future supervised learning
- **Modular Sub-Workflows** — Each workflow has a single responsibility (transform, insert, update, predict), making the system easy to maintain and extend

---

## Data Model (simplified)

```
caso_clinico ──── control_caso ──── predicciones
     │                 │
 paciente          semaforo
 diagnostico       observacion_original
 fase              json_filtrado_llm
```

Key entities: `caso_clinico`, `control_caso`, `predicciones`, `paciente`, `diagnostico`, `semaforo`, `fase`, `cia`, `distrito`

---

## Workflow Details

| Workflow | Description |
|---|---|
| `Población Inicial BD` | One-time migration of historical data from Excel to PostgreSQL |
| `TransformCasoFromExcel` | Cleans and structures a raw spreadsheet row into the DB schema |
| `Insertar/Actualizar Caso` | UPSERT logic — creates new case or adds new control record |
| `Actualizar Casos` | Main flow: receives webhook, detects new observation, routes to prediction |
| `Predicción Semáforo` | AI agent returns `{ color, justification, confidence }` structure |

---

## Setup

### Prerequisites

- Docker
- n8n (self-hosted or cloud)
- PostgreSQL instance (tested on Aiven)
- OpenAI API key
- Google Sheets with Apps Script configured

### Environment Variables

```env
DB_HOST=your_postgres_host
DB_PORT=5432
DB_NAME=medical_cases
DB_USER=your_user
DB_PASSWORD=your_password
OPENAI_API_KEY=your_key
N8N_WEBHOOK_URL=https://your-n8n-instance/webhook/...
```



## Challenges & Solutions

**Unstructured clinical text** — Medical notes contain abbreviations, style variations, and historical fragments mixed with new observations. Solved with a dedicated LLM agent that extracts only the semantic "novelty" from each update, significantly reducing noise in the database.

**Ambiguous classification boundary (yellow vs. red)** — Distinguishing between concerning and urgent evolution is non-trivial even for human doctors. Solved through iterative prompt engineering and validation cycles with the medical team until the client approved the MVP accuracy.

**Adoption friction** — Medical staff required minimal workflow disruption. Solved by keeping Google Sheets as the primary interface and adding automation as a transparent layer underneath.

---

## Results

- End-to-end functional pipeline: from Sheets input → AI prediction → structured DB → feedback loop
- Full clinical traceability across all weekly case controls
- Client-validated MVP with acceptable prediction accuracy
- Architecture ready for: advanced reporting dashboards, custom model fine-tuning on validated predictions, integration with HR/clinical management systems


## License

Private project — source code not publicly available. Architecture and documentation shared for portfolio purposes.
