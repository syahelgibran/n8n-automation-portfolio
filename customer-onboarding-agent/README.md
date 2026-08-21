# Customer Onboarding Agent — WarungOS

An automated onboarding system for **WarungOS**, a case study modeled
on real onboarding challenges faced by SaaS POS products in
Indonesia's F&B micro-business segment. Built entirely on n8n with
a multi-workflow architecture.

> **Impact:** Handles proactive follow-up and inbound support for
> the entire customer base simultaneously — no manual monitoring,
> no missed nudges, no CS queue for routine onboarding questions.

## What it does

Two independent workflows handle onboarding from both directions:

**Workflow 1 — Daily Decision Engine (Jalur A)**
Runs every morning at 09:00 WIB via Schedule trigger. Reads all
customer records from Google Sheets, calculates a churn risk score
per customer, and routes each one to the appropriate action — no
human involved.

| Action | Trigger condition |
|---|---|
| SKIP | Progressing on schedule |
| NUDGE | Stuck for 2+ days at current milestone |
| ESCALATE | Stuck for 5+ days |
| STOP | 3 nudges sent, all ignored |
| CLOSING | All 5 milestones completed |

**Workflow 2 — Inbound AI Agent (Jalur B)**
Triggered live when a registered customer sends a WhatsApp message.
An LLM agent answers product questions using RAG (pgvector semantic
search over the official WarungOS knowledge base). Escalation cases
are detected by keyword matching and forwarded to the CS team via
Telegram.

**Workflow 3 — Knowledge Base Ingest**
One-time manual run. Splits a 22-page PDF into chunks, embeds each
chunk using text-embedding-3-small, and stores vectors in Supabase
pgvector.

## Architecture

```
JALUR A
Schedule (09:00 WIB)
└── Read Google Sheets (Pelanggan + Progress_Milestone)
    └── Code: calculate churn score + routing decision
        ├── SKIP → no action
        ├── NUDGE → AI writes message → guardrail → WhatsApp
        ├── ESCALATE → AI writes message → guardrail → WhatsApp + Telegram alert
        ├── STOP → mark customer → Telegram alert
        └── CLOSING → AI writes message → guardrail → WhatsApp

JALUR B
WhatsApp Trigger
└── Normalize payload (handles 6 input formats)
    └── Lookup customer in Google Sheets
        ├── Unknown number → default reply → WhatsApp
        └── Known customer
            └── Build system prompt with customer context
                └── AI Agent (gpt-4o-mini)
                    ├── Tool: cari_panduan (Supabase pgvector RAG)
                    ├── Tool: cek_progress (Google Sheets read)
                    └── Output → guardrail → WhatsApp
                        └── Log to Google Sheets
                            └── Keyword check → Telegram (if escalation)
```

## Tech stack

| Layer | Tool |
|---|---|
| Orchestration | n8n (self-hosted Docker) |
| Operational DB | Google Sheets |
| Vector DB | Supabase pgvector |
| LLM | gpt-4o-mini via OpenRouter |
| Embeddings | text-embedding-3-small via OpenRouter |
| Messaging | Meta WhatsApp Cloud API |
| Notifications | Telegram Bot API |

## Key design decisions

- **Routing is deterministic, not AI.** Code decides who gets
  contacted, when, and how many times. The LLM only writes message
  content.
- **Guardrail on every AI output.** A validation node checks length,
  language, and format before any message is sent. Fallback to a safe
  default if the check fails.
- **RAG over prompt stuffing.** Product knowledge lives in Supabase,
  not the system prompt. Semantic search pulls only the relevant chunk
  per query.
- **One spreadsheet as the source of truth.** Pelanggan,
  Progress_Milestone, and Log_Interaksi tabs give a full operational
  view without a separate database.

## 6 verified scenarios

| # | Scenario | Workflow |
|---|---|---|
| 1 | Customer progressing on schedule → skipped | Jalur A |
| 2 | Stuck 2 days at milestone 2 → nudge sent | Jalur A |
| 3 | Stuck 5+ days → escalation + Telegram alert | Jalur A |
| 4 | 3 nudges ignored → STOP + CS handoff | Jalur A |
| 5 | Registered customer asks product question via WhatsApp → RAG agent answers | Jalur B |
| 6 | Customer message triggers escalation keyword → AI reply + Telegram alert to CS | Jalur B |

## Known Limitations

- **Google Sheets as operational DB** — suitable for small-to-medium
  customer bases (up to a few hundred active records). High-volume
  deployments would require a proper relational database.
- **RAG retrieval is single-turn** — the agent retrieves context once
  per query. Multi-turn conversations where context builds across
  messages are not currently handled.
- **Escalation detection is keyword-based** — the keyword list covers
  the most common cases but will miss paraphrased or implicit
  escalation signals.
- **WhatsApp payload normalisation covers 6 formats** — edge cases
  outside these formats will fall through to an unhandled state. No
  catch-all fallback is implemented beyond the unknown-number branch.
- **No re-entry logic after STOP** — a customer marked STOP requires
  manual intervention to re-enter the onboarding flow.

## License

Shared for portfolio and learning purposes. Free to use and modify.
