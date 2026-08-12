# Customer Onboarding Agent — WarungOS

An automated onboarding system for **WarungOS**, a fictional SaaS POS and inventory platform for F&B micro-businesses in Indonesia. Built entirely on n8n with a multi-workflow architecture.

## What it does

Two independent workflows handle onboarding from both directions:

**Workflow 1 — Daily Decision Engine (Jalur A)**
Runs every morning at 09:00 WIB via Schedule trigger. Reads all customer records from Google Sheets, calculates a churn risk score per customer, and routes each one to the appropriate action — no human involved.

| Action | Trigger condition |
|---|---|
| SKIP | Progressing on schedule |
| NUDGE | Stuck for 2+ days at current milestone |
| ESCALATE | Stuck for 5+ days |
| STOP | 3 nudges sent, all ignored |
| CLOSING | All 5 milestones completed |

**Workflow 2 — Inbound AI Agent (Jalur B)**
Triggered live when a registered customer sends a WhatsApp message. An LLM agent answers product questions using RAG (pgvector semantic search over the official WarungOS knowledge base). Escalation cases are detected by keyword matching and forwarded to the CS team via Telegram.

**Workflow 3 — Knowledge Base Ingest**
One-time manual run. Splits a 22-page PDF into chunks, embeds each chunk using text-embedding-3-small, and stores vectors in Supabase pgvector.

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

- **Routing is deterministic, not AI.** Code decides who gets contacted, when, and how many times. The LLM only writes message content.
- **Guardrail on every AI output.** A validation node checks length, language, and format before any message is sent. Fallback to a safe default if the check fails.
- **RAG over prompt stuffing.** Product knowledge lives in Supabase, not the system prompt. Semantic search pulls only the relevant chunk per query.
- **One spreadsheet as the source of truth.** Pelanggan, Progress_Milestone, and Log_Interaksi tabs give a full operational view without a separate database.

## 6 verified scenarios

| # | Scenario | Workflow |
|---|---|---|
| 1 | Customer progressing on schedule → skipped | Jalur A |
| 2 | Stuck 2 days at milestone 2 → nudge sent | Jalur A |
| 3 | Stuck 5+ days → escalation + Telegram | Jalur A |
| 4 | 3 nudges ignored → STOP + CS handoff | Jalur A |
| 5 | Customer asks product question via WhatsApp → RAG answer | Jalur B |
| 6 | All 5 milestones done → closing message | Jalur A |

## Known limitations

Honest notes on what this version does not handle yet.

**Proactive sends assume an open WhatsApp session.** Jalur A sends free-form text via the Cloud API. Meta only permits free-form messages within 24 hours of the customer's last inbound message — and Jalur A targets customers who have been inactive for 2+ days, so most nudges fall outside that window. Production use requires pre-approved Message Templates for every nudge, escalation, and closing message. The current build was verified against the sandbox number, not against the 24-hour policy.

**A failed WhatsApp send is still logged as delivered.** The send node runs with `onError: continueRegularOutput`, so the flow proceeds to `Catat Pesan ke Log` even when the API call fails. The anti-spam guard (`JEDA_NUDGE_HARI`) reads that same log, so a customer whose message silently failed is treated as already nudged and skipped on the next run. Delivery status is never read back.

**Escalation detection is regex, not comprehension.** Jalur B routes to CS on a fixed keyword list (`selisih|ganda|settlement|tidak bisa akses|reset password|bicara.*manusia|minta dihubungi|mencurigakan|tagihan|kontrak`). A customer who writes "kok uang masuknya beda ya sama struk" never matches and gets an ordinary RAG answer.

**The guardrail validates form, not truth.** Validation checks for empty output, length ceiling, and commercial-promise keywords (`gratis`, `diskon`, `refund`, `dijamin`, …). A fluent, correctly formatted, factually wrong answer passes untouched. There is no groundedness check against the retrieved chunk.

**Conversation memory is in-process and shallow.** Jalur B uses a window buffer of 6 turns keyed by `customer_id`. It lives in the n8n process, so every restart or container redeploy wipes mid-conversation context.

**Google Sheets is the scale ceiling.** Every daily run pulls the full `Pelanggan`, `Progress_Milestone`, and `Log_Interaksi` tabs into memory. This is comfortable at hundreds of customers and becomes a quota and latency problem in the low thousands. There are no transactions — Jalur A and Jalur B can interleave writes to `Log_Interaksi` under concurrency.

**The knowledge base is a one-shot manual ingest.** Updating the source PDF means clearing `warungos_kb` and re-running the whole workflow; there is no incremental re-index or chunk versioning. Chunking is fixed-size with 150-character overlap, so tables and step lists split across chunk boundaries retrieve poorly. Embeddings are pinned to 1536 dimensions.

**Indonesian only.** Prompts, guardrail keyword lists, and fallback copy are all Indonesian. A message in another language gets an Indonesian reply.

**Unknown numbers are a dead end.** Jalur B answers unrecognized senders with a static default and logs nothing — no lead capture, no routing to sales.

## How to deploy

> Resource IDs in these JSONs are placeholders (`YOUR_GOOGLE_SHEET_ID`, `YOUR_WHATSAPP_PHONE_NUMBER_ID`, `YOUR_TELEGRAM_CHAT_ID`, `YOUR_GOOGLE_DRIVE_FILE_ID`). Replace them with your own before running.

1. Import all three workflow JSONs into your n8n instance
2. Create a Google Sheets DB with tabs: `Pelanggan`, `Progress_Milestone`, `Log_Interaksi`
3. Set up a Supabase project, create table `warungos_kb` with pgvector, run the `match_warungos_kb` SQL function
4. Configure credentials: Google Sheets, OpenRouter, Supabase, WhatsApp Cloud API, Telegram Bot
5. Run the Knowledge Base Ingest workflow once with your PDF
6. Activate Jalur A and Jalur B
