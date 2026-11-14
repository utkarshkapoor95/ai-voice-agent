# AI Voice Agent - Dual-Mode Voice AI Showcase

A portfolio project demonstrating **two distinct voice AI capabilities** in a single system: an **inbound AI receptionist** (browser-based) and an **outbound lead qualification campaign** (telephony-based), powered by two different voice AI platforms, unified by n8n workflow automation.

🔗 **Live Showcase:** [voiceagent.autothynk.com](https://voiceagent.autothynk.com)

---

## What This Project Demonstrates

| Capability | Inbound Receptionist | Outbound Lead Qualifier |
|---|---|---|
| **Platform** | Vapi.ai (Web Widget) | Bolna.ai (Telephony) |
| **Agent Name** | Priya (SkinGlow Clinic) | Maya (AutoScale Solutions) |
| **Channel** | Browser-based voice chat | Outbound phone calls |
| **Use Case** | Answer questions, collect appointment details | Qualify leads, book discovery calls |
| **Trigger** | Visitor clicks mic on website | n8n workflow triggers campaign |
| **Cost Model** | ~$0.05/min (Vapi platform + STT/LLM/TTS) | ~$0.06/min (Bolna bundled rate) |

---

## Architecture

### System Overview

```mermaid
flowchart TB
    subgraph INBOUND["🎙️ INBOUND - Clinic Receptionist"]
        V1[Website Visitor] -->|Clicks mic widget| V2[Vapi Web Widget]
        V2 -->|WebRTC audio stream| V3[Vapi Cloud]
        V3 -->|STT + LLM + TTS| V4["Agent: Priya<br/>SkinGlow Clinic"]
        V4 -->|Appointment details| V5[n8n Webhook]
        V5 --> V6[Google Sheets Log]
        V5 --> V7[Telegram Notification]
    end

    subgraph OUTBOUND["📞 OUTBOUND - Lead Qualification"]
        O1[Google Sheets<br/>Lead List] -->|Webhook trigger| O2[n8n Workflow]
        O2 -->|Telegram approval gate| O3{Approved?}
        O3 -->|No| O4[Skip - Not Authorized]
        O3 -->|Yes| O5[Bolna.ai API]
        O5 -->|Outbound phone call| O6["Agent: Maya<br/>AutoScale Solutions"]
        O6 -->|Call complete| O7[Bolna Get Status]
        O7 --> O8{Picked Up?}
        O8 -->|Yes| O9[Groq LLM<br/>Classify Intent]
        O8 -->|No| O10{2nd Attempt?}
        O9 --> O11{Interested?}
        O11 -->|Yes| O12["✅ Interested<br/>Email + Sheet + Telegram"]
        O11 -->|No| O13["❌ Not Interested<br/>Email + Sheet"]
        O10 -->|No| O14["📞 No Answer<br/>Email + Sheet + Retry"]
        O10 -->|Yes| O15["💀 Dead Contact<br/>Email + Sheet + Stop"]
    end

    subgraph SHARED["🔧 SHARED INFRASTRUCTURE"]
        S1[n8n on DigitalOcean]
        S2[Telegram Bot - Approval Gate]
        S3[Google Sheets - Tracking]
        S4[Brevo - Transactional Email]
        S5[Groq - Intent Classification]
    end

    V5 -.-> S1
    O2 -.-> S1
    O3 -.-> S2
    V6 -.-> S3
    O12 -.-> S3
    O12 -.-> S4
    O13 -.-> S4
    O14 -.-> S4
    O15 -.-> S4
    O9 -.-> S5
```

### Outbound Campaign - Detailed Node Flow

```mermaid
flowchart LR
    A[Schedule Trigger<br/>or Webhook] --> B[Get Leads<br/>from Sheet]
    B --> C[Loop Over<br/>Each Lead]
    C --> D{Skip if<br/>Called Recently?}
    D -->|Skip| C
    D -->|Proceed| E[Clean Phone<br/>E.164 Format]
    E --> F[Bolna API<br/>Place Call]
    F --> G[Wait 120s]
    G --> H[Bolna API<br/>Get Status]
    H --> I{Transcript<br/>Exists?}
    I -->|Yes: Picked Up| J[Groq LLM<br/>Analyze Intent]
    J --> K{Interested?}
    K -->|Yes| L[Brevo: Opportunity Email<br/>Sheet: Mark Interested<br/>Telegram: Alert Owner]
    K -->|No| M[Brevo: Reassurance Email<br/>Sheet: Mark Not Interested]
    I -->|No: Not Picked Up| N{Attempt<br/>Count >= 1?}
    N -->|No: 1st Fail| O[Brevo: No Answer Email<br/>Sheet: Mark No Answer<br/>Retry in 3 Days]
    N -->|Yes: 2nd Fail| P[Brevo: Dead Contact Email<br/>Sheet: Mark Dead<br/>No More Retries]
    L --> C
    M --> C
    O --> C
    P --> C
```

---

## Tech Stack

| Component | Technology | Role |
|---|---|---|
| Inbound Voice AI | [Vapi.ai](https://vapi.ai) | Browser-based voice widget, WebRTC |
| Outbound Voice AI | [Bolna.ai](https://bolna.ai) | Telephony API, outbound phone calls |
| Workflow Engine | [n8n](https://n8n.io) (self-hosted) | Orchestration, branching logic, API calls |
| LLM (Classification) | [Groq](https://groq.com) (LLama 3.3 70B) | Post-call transcript intent classification |
| Email | [Brevo](https://brevo.com) | Transactional follow-up emails (4 templates) |
| Data Store | Google Sheets | Lead tracking, call status, attempt counts |
| Notifications | Telegram Bot API | Owner alerts + demo approval gate |
| Hosting | DigitalOcean | n8n droplet |
| Showcase Page | GitHub Pages | Static landing page |

---

## Project Structure

```
ai-voice-agent/
├── README.md
├── index.html                              # Showcase landing page
├── prompts/
│   ├── inbound_receptionist.md             # Vapi agent: Priya (SkinGlow Clinic)
│   └── outbound_qualifier.md               # Bolna agent: Maya (AutoScale Solutions)
├── workflows/
│   ├── outbound_campaign_SANITIZED.json    # Production n8n workflow (credentials stripped)
│   ├── outbound_demo.json                  # Demo workflow skeleton (single-lead mode)
│   ├── inbound_webhook.json                # Inbound call data handler
│   └── telegram_gate.json                  # Telegram approval mechanism
└── docs/
    └── assets/                             # Screenshots, diagrams
```

---

## How It Works

### Inbound Flow (Vapi - Browser Voice)

1. Visitor lands on the clinic website and clicks the mic widget
2. Vapi opens a WebRTC audio stream to its cloud
3. The AI agent (Priya) greets the visitor and handles their query
4. If the visitor wants to book an appointment, Priya collects their details
5. Call data is sent via webhook to n8n
6. n8n logs the interaction to Google Sheets and sends a Telegram notification

### Outbound Flow (Bolna - Phone Calls)

1. A lead list sits in Google Sheets with names, phone numbers, and status
2. The n8n workflow is triggered (manually or via schedule)
3. **Telegram Gate:** A message is sent to the owner's Telegram asking for approval before any call is placed
4. On approval, the workflow loops through each lead:
   - Skips leads already called recently (3-day cooldown for No Answer)
   - Cleans the phone number to E.164 international format
   - Places an outbound call via Bolna's API
   - Waits 120 seconds for the call to complete
   - Polls Bolna for the call transcript and status
5. **If picked up:** Transcript is sent to Groq (LLama 3.3 70B) for intent classification
   - **Interested** -> Opportunity email + Google Sheets update + Telegram alert
   - **Not Interested** -> Reassurance email + Google Sheets update
6. **If not picked up:**
   - **1st failure** -> "We tried calling" email + mark for retry in 3 days
   - **2nd failure** -> "Door is open" final email + mark as Dead Contact

### Security: Telegram Approval Gate

Both demos are gated behind a Telegram approval mechanism. When triggered:
1. n8n sends a message to the owner's Telegram with call details
2. Owner replies "yes" to approve or "no" to block
3. Only approved calls proceed, preventing unauthorized credit burn

---

## Production Background

The outbound workflow architecture is based on a **production system** that made **5,000+ outbound calls** with an **~8% positive response rate** (~400 qualified leads) during a real client engagement. The production system used Airtable as the data store and ran daily scheduled campaigns.

For this portfolio showcase, the system has been adapted with:
- Google Sheets replacing Airtable (simpler setup, no paid plan needed)
- Telegram approval gate added (demo security)
- Both Vapi inbound and Bolna outbound unified in one project
- All credentials stripped and replaced with placeholders

---

## Setup Guide

### Prerequisites
- [Vapi.ai](https://vapi.ai) account (free tier: $10 credits)
- [Bolna.ai](https://bolna.ai) account (free tier: $5 credits)
- [n8n](https://n8n.io) instance (self-hosted or cloud)
- [Groq](https://groq.com) API key (free tier available)
- [Brevo](https://brevo.com) account (free tier: 300 emails/day)
- Telegram Bot (via [@BotFather](https://t.me/BotFather))
- Google Sheets API credentials

### Quick Start

1. **Clone this repo:**
   ```bash
   git clone https://github.com/utkarshkapoor95/ai-voice-agent.git
   ```

2. **Set up Vapi Agent:**
   - Create a new assistant in Vapi dashboard
   - Paste the prompt from `prompts/inbound_receptionist.md`
   - Configure voice and model settings
   - Copy the Assistant ID

3. **Set up Bolna Agent:**
   - Create a new agent in Bolna dashboard
   - Paste the prompt from `prompts/outbound_qualifier.md`
   - Configure voice provider and LLM
   - Copy the Agent ID

4. **Import n8n Workflows:**
   - Import `workflows/outbound_demo.json` into your n8n instance
   - Replace all `YOUR_*` placeholders with your actual credentials
   - Set up the Telegram gate with your bot token and chat ID

5. **Deploy Showcase Page:**
   - Push to GitHub
   - Enable GitHub Pages
   - Set up CNAME for `voiceagent.autothynk.com`

---

## Cost Analysis

| Component | Free Tier | Per-Minute Cost |
|---|---|---|
| Vapi.ai | $10 credits | ~$0.05/min (platform + Deepgram STT + GPT-4o + ElevenLabs TTS) |
| Bolna.ai | $5 credits | ~$0.06/min (bundled: platform + STT + LLM + TTS + telephony) |
| Groq | Free tier | Free (for classification - minimal tokens) |
| Brevo | 300 emails/day | Free |
| n8n | Self-hosted | $0 (runs on existing DigitalOcean droplet) |

**Estimated demo cost:** ~$0.30-0.50 per complete demo cycle (1 inbound + 1 outbound call)

---

## Author

**Utkarsh Kapoor**
- MBA @ ABV-IIITM Gwalior | AI Automation & Data Systems Builder
- [Portfolio](https://utkarshkapoor.com) | [LinkedIn](https://linkedin.com/in/utkarsh-kapoor-618256203) | [GitHub](https://github.com/utkarshkapoor95)

---

## License

This project is open-sourced for portfolio and educational purposes. The workflow architecture and prompts are original work. Feel free to use as reference for your own voice AI projects.
