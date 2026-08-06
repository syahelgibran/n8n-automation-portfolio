# n8n Automation Portfolio

Workflow automation systems built with n8n — designed for real 
business problems, not tutorials.

Each project here was built for an actual client or use case, 
deployed, and reviewed by someone who had to use it. The focus 
is on systems that work reliably when handed to non-technical 
users: clear logic, documented decisions, and honest notes on 
what the current version doesn't handle yet.

## Projects

| Project | What it does | Stack |
|---|---|---|
| [AI CV Screener](./cv-screener-ai/) | Reads incoming CVs, evaluates against job qualifications using an LLM, writes structured results to Sheets. Decision logic is in code — not the AI. | n8n · OpenRouter · LangChain · Google Sheets |
| WhatsApp Order System *(coming soon)* | State machine that handles customer orders end-to-end via WhatsApp — from first message to payment confirmation | n8n · WhatsApp Business API |
| HR Attendance Recap *(coming soon)* | Daily automated lateness report with per-employee summary, fine calculation, and Telegram notification to HR | n8n · Google Sheets · Telegram |

## Tech I work with

**Workflow automation:** n8n (self-hosted & cloud)  
**AI / LLM:** OpenRouter · OpenAI API · LangChain · prompt engineering · structured output  
**Integration:** REST API · webhook · OAuth 2.0 · Google Workspace  
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

I'm Syahel — an automation specialist based in East Java, 
Indonesia. My background is in agribusiness and entrepreneurship 
(Universitas Brawijaya), not computer science. I got into 
automation because I needed it: running actual businesses made 
the cost of manual, repetitive work very obvious very fast.

I now build automation systems for SMEs — primarily in 
Indonesia, with growing interest in working with international 
clients remotely.

**Open to:** freelance projects · automation consulting · 
remote collaboration

📧 syahelgibran11@gmail.com  
💼 Syahel Gibran Azrofie
