# n8n Automation Portfolio

Workflow automation systems built with n8n — designed for real
business problems, not tutorials.

Each project here is designed around a real business problem — the
kind that repeats across SMEs and operations teams. Built to be
deployable, not just demonstrable.

## Projects

| Project | What it does | Stack |
|---|---|---|
| [AI CV Screener](./cv-screener-ai/) | Reads incoming CVs, evaluates against job qualifications using an LLM, writes structured results to Sheets. Decision logic is in code — not the AI. | n8n · OpenRouter · LangChain · Google Sheets |
| [Customer Onboarding Agent](./customer-onboarding-agent/) | Multi-workflow onboarding system for a SaaS POS product (case study). Deterministic daily churn-risk routing plus an inbound RAG agent on WhatsApp. Routing is code; the LLM only writes copy. | n8n · OpenRouter · Supabase pgvector · WhatsApp Cloud API · Telegram |

## Tech I work with

**Workflow automation:** n8n (self-hosted & cloud)  
**AI / LLM:** OpenRouter · OpenAI API · LangChain · prompt engineering · structured output  
**Integration:** REST API · webhook · OAuth 2.0 · Google Workspace · Supabase  
**Notification:** Telegram Bot · WhatsApp Business API  
**Storage:** Google Sheets · Google Drive  

## Approach

A few things I consistently apply across every system:

**Results before credentials.** I come from a business background
(ran a fashion business, led 4 business units as a student
organization chairman) before moving into automation. Every
workflow I build gets evaluated by one question first: what does
this save, and for whom?

**Code decides, AI reads.** In any workflow where an LLM is
involved, I keep decision logic deterministic and in code. AI
handles interpretation; code handles rules. This makes results
auditable, consistent, and easy to adjust without prompt
engineering.

**Ship with known limitations documented.** Every README in
this repo includes a Known Limitations section. Systems that
pretend to have no edge cases just haven't been run with real
data yet.

## Background

I'm Syahel — an AI Automation Specialist based in Jakarta,
Indonesia. My background is in agribusiness and entrepreneurship
(Universitas Brawijaya, GPA 3.45). I got into automation because
I needed it: running actual businesses made the cost of manual,
repetitive work very obvious very fast.

I now build automation systems focused on SME operations and
business workflows — with growing interest in working with
international clients remotely.

**Open to:** full-time roles · freelance projects · automation consulting ·
remote collaboration

📧 syahelgibran11@gmail.com  
💼 [linkedin.com/in/syahel](https://linkedin.com/in/syahel)  
💻 Syahel Gibran Azrofie
