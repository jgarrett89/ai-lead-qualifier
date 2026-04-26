# AI Lead Qualification Agent

Automatically scores and routes inbound leads using Claude AI, n8n, and GoHighLevel.

## What it does

A contact fills out a GHL form. The form submission hits an n8n webhook. Claude scores the lead 1-10 and returns a tier (hot/warm/cold), a reason, and a suggested next action. Hot leads (7+) fire a Slack alert. Every lead gets logged to Google Sheets.

The whole process runs in under 5 seconds.

## Stack

- **n8n** — workflow orchestration (self-hosted on Oracle Cloud)
- **GoHighLevel** — form capture and CRM webhook trigger
- **Claude API** (claude-haiku-4-5) — lead scoring and qualification
- **Slack** — hot lead alerts
- **Google Sheets** — lead database

## How it works

```
GHL Form Submission
      ↓
n8n Webhook
      ↓
Edit Fields (normalize GHL field names)
      ↓
HTTP Request → Claude API (score + qualify)
      ↓
Code Node (parse Claude JSON response)
      ↓
If Node (score >= 7?)
     ↙         ↘
  TRUE         FALSE
Slack alert   (skip)
     ↘         ↙
  Google Sheets (log all leads)
```

## Scoring logic

Claude scores each lead on a 1-10 scale:

- **8-10 (Hot)** — clear need, decision-maker, specific ask, urgency
- **5-7 (Warm)** — some interest but vague or early stage
- **1-4 (Cold)** — poor fit, no real intent, or missing information

## Setup

### Prerequisites

- n8n instance (self-hosted or cloud)
- Anthropic API key
- GoHighLevel account with a form
- Slack workspace with a bot token
- Google Sheets with the following headers: `Timestamp | Name | Email | Business | Message | Score | Tier | Reason | Suggested Action`

### Installation

1. Import `lead-qualifier.json` into n8n
2. Add your credentials:
   - Claude API key in the HTTP Request node header `x-api-key`
   - Google Sheets OAuth credential
   - Slack bot token
3. Update the Slack channel name in the Slack node
4. Set your Google Sheet ID in the Google Sheets node
5. Copy your n8n webhook URL
6. In GHL, create an Automation workflow triggered by your form submission with a Webhook action pointing to your n8n URL
7. Activate the workflow in n8n

### GHL field mapping

GHL sends form fields using these keys. Make sure your form fields match:

| Form Field | GHL Key |
|---|---|
| Name | `Name` |
| Email | `email` |
| Business Name | `Business Name` |
| Message | `Message` |
| Source | `contact_source` |

## Demo

[Add Loom video link here]

## Example output

**Slack alert (hot lead):**
```
🔥 New Hot Lead — HOT (8/10)
───────────────────────────
👤 Sarah Chen
🏢 Glow Aesthetics Med Spa
📧 sarah@medspa-pro.com

💬 "We just bought an InMode Morpheus8 and need $150K to fund our expansion."

🧠 Why: Established med spa with recent capital investment and clear funding need
⚡ Action: Schedule discovery call within 24 hours
```

**Google Sheets row:**

| Timestamp | Name | Email | Business | Score | Tier | Reason |
|---|---|---|---|---|---|---|
| 2026-04-26T14:23:11 | Sarah Chen | sarah@medspa-pro.com | Glow Aesthetics | 8 | hot | Clear need, specific ask, urgency |

## Infrastructure

- Oracle Cloud free tier (VM.Standard.E2.1.Micro, Ubuntu 22.04)
- Docker
- Caddy reverse proxy with automatic SSL (Let's Encrypt)
- n8n v2.17.7

## Author

Built by Jimmie as part of an AI automation portfolio targeting roles in AI engineering and workflow automation.
