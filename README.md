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
| **Trigger** | Visitor clicks mic on website | Web form triggers n8n workflow |
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
        O1[Web Form] -->|Webhook trigger| O2[n8n Workflow]
        O2 --> O5[Bolna.ai API]
        O5 -->|Outbound phone call| O6["Agent: Maya<br/>AutoScale Solutions"]
        O6 -->|Call complete| O7[Bolna Get Status]
        O7 --> O8{Picked Up?}
        O8 -->|Yes| O9[Groq LLM<br/>Classify Intent]
        O8 -->|No| O10[Mark No Answer]
        O9 --> O11{Interested?}
        O11 -->|Yes| O12["Interested<br/>Sheet + Telegram Alert"]
        O11 -->|No| O13["Not Interested<br/>Sheet Log"]
    end

    subgraph SHARED["🔧 SHARED INFRASTRUCTURE"]
        S1[n8n on DigitalOcean]
        S2[Telegram Bot - Owner Alerts]
        S3[Google Sheets - Tracking]
        S5[Groq - Intent Classification]
    end

    V5 -.-> S1
    O2 -.-> S1
    O12 -.-> S2
    V6 -.-> S3
    O12 -.-> S3
    O13 -.-> S3
    O9 -.-> S5
```

### Outbound Demo - Node Flow

```mermaid
flowchart LR
    A[Webhook<br/>Form Submit] --> B[Bolna API<br/>Place Call]
    B --> C[Wait 120s]
    C --> D[Bolna API<br/>Get Status]
    D --> E{Transcript<br/>Exists?}
    E -->|Yes: Picked Up| F[Groq LLM<br/>Classify Intent]
    F --> G[Google Sheets<br/>Log Result]
    G --> H[Telegram<br/>Alert Owner]
    E -->|No: Not Picked Up| I[Google Sheets<br/>Log No Answer]
    I --> H
```

---

## Tech Stack

| Component | Technology | Role |
|---|---|---|
| Inbound Voice AI | [Vapi.ai](https://vapi.ai) | Browser-based voice widget, WebRTC |
| Outbound Voice AI | [Bolna.ai](https://bolna.ai) | Telephony API, outbound phone calls |
| Workflow Engine | [n8n](https://n8n.io) (self-hosted) | Orchestration, branching logic, API calls |
| LLM (Classification) | [Groq](https://groq.com) (openai/gpt-oss-120b) | Post-call transcript intent classification |
| Data Store | Google Sheets | Lead tracking, call status, classification |
| Notifications | Telegram Bot API | Owner alerts (post-call summaries) |
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
│   └── outbound_demo.json                  # Demo workflow (single-lead mode)
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

1. User submits the demo form with their name, phone number, and scenario selection
2. n8n webhook receives the form data and triggers the workflow
3. Bolna API places a live outbound call to the submitted phone number
4. The AI agent conducts the conversation based on the selected scenario
5. After the call completes, n8n polls Bolna for the transcript and status
6. **If picked up:** Transcript is sent to Groq (openai/gpt-oss-120b) for intent classification
   - Result and transcript are logged to Google Sheets
   - Telegram sends the owner a post-call summary with classification
7. **If not picked up:** Logged as "No Answer" in Google Sheets

---

## Production Background

The outbound workflow architecture is based on a **production system** that made **5,000+ outbound calls** with an **~8% positive response rate** (~400 qualified leads) during a real client engagement. The production system used Airtable as the data store and ran daily scheduled campaigns.

For this portfolio showcase, the system has been adapted with:
- Google Sheets replacing Airtable (simpler setup, no paid plan needed)
- Both Vapi inbound and Bolna outbound unified in one project
- Telegram for post-call owner notifications
- All credentials stripped and replaced with placeholders

---

## Setup Guide

### Prerequisites
- [Vapi.ai](https://vapi.ai) account (free tier: $10 credits)
- [Bolna.ai](https://bolna.ai) account (free tier: $5 credits)
- [n8n](https://n8n.io) instance (self-hosted or cloud)
- [Groq](https://groq.com) API key (free tier available)
- Telegram Bot (via [@BotFather](https://t.me/BotFather)) for owner alerts
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
   - Set up the Telegram bot with your bot token and chat ID for notifications

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
