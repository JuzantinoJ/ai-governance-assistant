# AI Governance & Incident Assistant

> An enterprise-grade AI workflow built on [n8n](https://n8n.io) that automatically classifies, summarizes, extracts action items from, and responds to security incidents, audit findings, and governance events.

---

## What This Project Does

Paste any unstructured security or governance finding as text input. The system will:

1. **Validate and sanitize** the input (injection guard, length check)
2. **Classify** the finding — type and severity (CRITICAL / HIGH / MEDIUM / LOW / INFO)
3. **Summarize** it in 2–3 professional sentences
4. **Extract action items** with ownership and priority
5. **Draft and send** a professional email response
6. **Store all results** to a Google Sheet with a full audit trail

All of this happens automatically, in seconds, without any manual work.

---

## Supported Input Types

| Type                    | Example                                                       |
| ----------------------- | ------------------------------------------------------------- |
| Security incident       | "CPU at 98% for 20 minutes, possible cryptominer"             |
| Vulnerability finding   | "CVE-2024-XXXX found in firewall firmware v2.1.4"             |
| Governance / compliance | "SOC 2 control AC-3 not implemented for admin accounts"       |
| Audit finding           | "Access logs not retained beyond 30 days per policy"          |
| SSP finding             | "System boundary not documented for production environment"   |
| Vendor response         | "Third-party vendor has not completed annual security review" |
| Patch update            | "Critical patch MS24-001 pending on 14 endpoints"             |
| Operational ticket      | "MFA bypass reported by user on VPN login"                    |

---

## Architecture Overview

```
User Input (text)
      ↓
Webhook Trigger
      ↓
Input Validator  ←─── Sanitize · Length check · Injection guard
      ↓
AI: Classify     ←─── Type · Severity · Confidence score
      ↓
AI: Summarize    ←─── 2–3 sentence factual brief
      ↓
AI: Extract Actions ← Structured JSON with owner + priority
      ↓
Output Validator ←─── Schema check · Confidence gate · Fallback handler
      ↓
AI: Draft Email
      ↓
Send Email (Gmail / SMTP)
      ↓
Store to Google Sheets ← Full audit row with run_id + timestamp
      ↓
Respond to User (JSON)
```

For the full layered architecture, see [`ARCHITECTURE.md`](./ARCHITECTURE.md).

---

## Tech Stack

| Layer                  | Technology                                     |
| ---------------------- | ---------------------------------------------- |
| Workflow orchestration | [n8n](https://n8n.io) (self-hosted via Docker) |
| AI model               | OpenAI GPT-4o or Anthropic Claude              |
| Database (V1)          | Google Sheets                                  |
| Database (V2 target)   | Supabase (PostgreSQL)                          |
| Notifications          | Gmail / SMTP                                   |
| Containerization       | Docker + Docker Compose                        |

---

## Project Structure

```
ai-governance-assistant/
│
├── docker-compose.yml             ← Run n8n locally with Docker
├── .env.example                   ← Environment variable template
├── .gitignore
├── README.md                      ← This file
├── ARCHITECTURE.md                ← Full system design and decisions
│
├── workflows/
│   ├── v1-main-workflow.json      ← Main n8n workflow (export)
│   └── v1-error-handler.json      ← Error sub-workflow (export)
│
├── prompts/
│   ├── classify-v1.txt            ← Classification prompt
│   ├── summarize-v1.txt           ← Summarization prompt
│   ├── extract-actions-v1.txt     ← Action item extraction prompt
│   └── draft-email-v1.txt         ← Email draft prompt
│
├── docs/
│   ├── node-guide.md              ← Node-by-node explanation
│   └── schema.md                  ← Google Sheets column schema
│
└── tests/
    ├── sample-incident.txt
    ├── sample-vulnerability.txt
    ├── sample-audit-finding.txt
    ├── sample-governance.txt
    ├── sample-ssp.txt
    ├── sample-vendor.txt
    ├── sample-patch.txt
    └── sample-ticket.txt
```

---

## Getting Started

### Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
- An OpenAI or Anthropic API key
- A Google account (for Sheets + Gmail integration)

### 1. Clone the repo

```bash
git clone https://github.com/JuzantinoJ/ai-governance-assistant.git
cd ai-governance-assistant
```

### 2. Set up your environment variables

```bash
cp .env.example .env
```

Open `.env` and fill in your values. See `.env.example` for descriptions of each variable.

> **Never commit your `.env` file.** It is already in `.gitignore`.

### 3. Start n8n with Docker

```bash
docker compose up -d
```

n8n will be available at: **http://localhost:5678**

First-time setup: n8n will prompt you to create an admin account on first visit.

### 4. Import the workflow

1. Open n8n at http://localhost:5678
2. Go to **Workflows → Import from file**
3. Select `workflows/v1-main-workflow.json`
4. Add your credentials (OpenAI, Gmail, Google Sheets) under **Credentials**
5. Activate the workflow

### 5. Test it

```bash
curl -X POST http://localhost:5678/webhook-test/incident-intake \
  -H "Content-Type: application/json" \
  -d '{"text": "Critical vulnerability CVE-2024-9999 found in firewall firmware. Unauthenticated RCE possible. Affects all devices running version < 3.2.1"}'
```

You should receive a JSON response with severity, summary, and action items.

---

## Environment Variables

See `.env.example` for the full list. Key variables:

| Variable             | Description                                               |
| -------------------- | --------------------------------------------------------- |
| `POSTGRES_USER`      | PostgreSQL username for n8n                               |
| `POSTGRES_PASSWORD`  | PostgreSQL password — change before first run             |
| `POSTGRES_DB`        | Database name                                             |
| `N8N_ENCRYPTION_KEY` | Encrypts credentials at rest — **set once, never change** |
| `GENERIC_TIMEZONE`   | Your timezone e.g. `Asia/Kuala_Lumpur`                    |
| `N8N_HOST`           | Hostname n8n runs on (default: `localhost`)               |

---

## Implementation Roadmap

| Phase                       | Status         | Description                          |
| --------------------------- | -------------- | ------------------------------------ |
| Phase 1 — Foundation        | ✅ In progress | Webhook + input validation           |
| Phase 2 — AI Core           | ⬜ Planned     | Classify · Summarize · Extract       |
| Phase 3 — Outputs & Storage | ⬜ Planned     | Email · Sheets · Response            |
| Phase 4 — Hardening         | ⬜ Planned     | Error handling · Retry · Audit trail |

---

## Security Notes

- All secrets are managed via environment variables — **nothing is hardcoded**
- User input is sanitized before it reaches the AI model
- AI output is validated before any action is taken
- The `.env` file and any `*.key` files are excluded from git via `.gitignore`
- Webhook secret header (`X-Webhook-Secret`) will be added in Phase 4

---

## What I'm Learning

This project is a hands-on study in:

- n8n workflow design and node architecture
- AI orchestration and prompt engineering
- Enterprise workflow patterns (modular design, error handling, audit trails)
- API integration (OpenAI / Anthropic, Google Sheets, Gmail)
- Security-first automation habits
- Docker-based self-hosting

---

## Future Plans (V2 / V3)

- Migrate storage to Supabase (PostgreSQL)
- Add Slack alerts for CRITICAL severity findings
- Add per-input-type routing via Switch node
- Add RAG layer using T1N0 governance documentation
- Add JWT authentication on the webhook endpoint
- Deploy local AI model via Ollama for sensitive data

---

## Related

- [n8n-setup-macOS](https://github.com/JuzantinoJ/n8n-setup-macOS) — How to install and run n8n on macOS (start here if you're new to n8n)

---

## Author

**JuzantinoJ** — building AI systems, automation workflows, and enterprise AI infrastructure.

> This project is part of my transition into AI systems engineering, cloud automation, and enterprise AI governance.
