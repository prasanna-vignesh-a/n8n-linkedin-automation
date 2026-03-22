# 🤖 Autonomous LinkedIn Content Engine — n8n Agentic Workflow

> Architected an end-to-end agentic workflow that autonomously generated, validated, and published LinkedIn posts for **80+ consecutive days — zero manual intervention.**

---

## 📌 Overview

This project automates the entire LinkedIn content lifecycle using **n8n** as the orchestration layer. Every day at 9:30 PM IST, the workflow picks up the next topic from a Google Sheet, generates a formatted LinkedIn post using **Google Gemini 2.5 Flash**, cleans the output, and publishes it directly via the LinkedIn API — fully unattended.

**Result:** 80+ consecutive days of consistent, high-quality LinkedIn presence with no human involvement.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│              SCHEDULER (Daily)              │
│           Cron trigger fires every day                  │
└───────────────────────┬─────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│           FETCH TOPIC (Google Sheets)                   │
│   Reads first row where Post Status = "Pending"         │
└───────────────────────┬─────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│           CONTENT GENERATION (Google Gemini)            │
│   Gemini 2.5 Flash → formats post with emojis,          │
│   post with emojis, sections, hashtags                  │
└───────────────────────┬─────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│                 CLEAN OUTPUT                            │
│   Strips markdown, escapes quotes & newlines            │
│   → publish-ready plain text                            │
└───────────────────────┬─────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│              AUTO-PUBLISH (LinkedIn API)                 │
│     HTTP POST → api.linkedin.com/v2/ugcPosts            │
└───────────────────────┬─────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│           LOGGING & CLEANUP                             │
│   Deletes used row from Google Sheets                   │
└───────────────────────┬─────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│           GMAIL NOTIFICATION                            │
│   Sends "Post has been Published" email confirmation    │
└─────────────────────────────────────────────────────────┘
```

---

## 🖼️ Workflow Canvas

![n8n Workflow Overview](screenshots/workflow-overview.png)

---

## ⚙️ Tech Stack

| Layer | Tool |
|---|---|
| Workflow Orchestration | n8n |
| LLM | Google Gemini 2.5 Flash (`gemini-2.5-flash-preview-04-17`) |
| Output Processing | n8n Code node (JavaScript) |
| Publishing | LinkedIn UGC API (`/v2/ugcPosts`) |
| Content Queue | Google Sheets (`Post Status: Pending`) |
| Notifications | Gmail |
| Scheduling | n8n Cron — daily |

---

## 🔄 Workflow Breakdown

### 1. Daily Schedule
- Fires every day
- Kicks off the full pipeline automatically

### 2. Fetch Topic from Sheet
- Reads the first row from Google Sheets where `Post Status = Pending`
- Each row contains a `Day` number and `Topic` for that day's post
- Uses `returnFirstMatch` to always pick the next pending topic

### 3. Generate LinkedIn Post (Gemini 2.5 Flash)
- n8n AI Agent node powered by Google Gemini 2.5 Flash
- System prompt: *"You are a Power Platform expert writing daily LinkedIn posts"*
- Generates a fully formatted post with emojis, sections, and hashtags
- Follows a consistent format with day number, topic, emojis, and hashtags

### 4. Format Post Output
- JavaScript code node that post-processes the LLM output
- Replaces `**bold**` markdown with `🔹 text`
- Escapes newlines and double quotes for clean JSON delivery to LinkedIn API

### 5. Publish to LinkedIn
- HTTP POST to `api.linkedin.com/v2/ugcPosts`
- Publishes as a public UGC post on the LinkedIn profile
- Uses OAuth2 credentials for authentication

### 6. Remove Used Topic
- Deletes the published row from Google Sheets
- Keeps the queue clean and prevents any topic from being posted twice

### 7. Send Success Notification
- Gmail sends a confirmation email after every successful publish
- Instant visibility — no need to manually check if the workflow ran

---

## 📁 Repository Structure

```
n8n-linkedin-automation/
├── README.md
├── flows/
│   └── linkedin-automation.json          # Full workflow export (sanitized)
└── screenshots/
    └── workflow-overview.png             # n8n canvas screenshot
```

---

## 🚀 How to Import & Run

1. Start n8n using Docker:
   ```bash
   docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
   ```

2. Open `http://localhost:5678` in your browser

3. Go to **Workflows → Import from file**

4. Import `linkedin-automation.json` from the `/flows` folder

5. Add your credentials in n8n:
   - Google Gemini API key
   - LinkedIn OAuth token
   - Google Sheets OAuth (Google account)
   - Gmail OAuth (Google account)

6. Set up your Google Sheet with columns: `Day`, `Topic`, `Post Status`
   - Set `Post Status = Pending` for each row

7. Activate the workflow — it runs every day automatically

---

## 💡 Key Design Decisions

- **Google Sheets as a self-managed content queue** — topics stored as rows, filtered by `Post Status = Pending`, deleted after use — no row ever gets posted twice
- **Gemini 2.5 Flash for generation** — fast, cost-efficient, and produces consistently formatted LinkedIn posts with emojis and structure
- **CleanOutput code node** — bridges the gap between LLM markdown output and LinkedIn's plain-text API requirement; converts `**bold**` to emoji bullets and escapes special characters
- **Daily 9:30 PM IST schedule** — timed for peak LinkedIn engagement in the Indian audience timezone

---

## 📊 Outcome

| Metric | Result |
|---|---|
| Consecutive days automated | 80+ |
| Manual interventions required | 0 |
| Human approval steps | None |
| Consistency | 100% (no missed days) |

---

## 🛠️ Built With

- [n8n](https://n8n.io) — open-source workflow automation
- [Google Gemini API](https://ai.google.dev) — Gemini 2.5 Flash for content generation
- [LinkedIn UGC API](https://developer.linkedin.com) — auto-publishing posts
- [Google Sheets API](https://developers.google.com/sheets) — content queue management
- [Gmail API](https://developers.google.com/gmail) — publish confirmation notifications

---

## ⚠️ Setup Notes

- Replace `YOUR_LINKEDIN_TOKEN_HERE` in the HTTP Request node with your LinkedIn Bearer token
- Replace `YOUR_LINKEDIN_PERSON_ID` in the request body with your LinkedIn URN (e.g. `urn:li:person:xxxxxxxx`)
- Replace `YOUR_GOOGLE_SHEET_ID` with your actual Google Sheet ID
- Replace `YOUR_EMAIL@gmail.com` with your Gmail address

---

*Part of my automation portfolio. Built to solve a real consistency problem — and it worked.*
