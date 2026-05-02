# Decoupled Agentic Lead Pipeline

A modular n8n lead-generation system for collecting prospects, enriching them with AI, sending personalized outreach, tracking opens, monitoring replies, and surfacing failures.

![Architecture diagram](docs/architecture-diagram.png)

## What it does

This project is split into small workflows so each part can be edited, tested, and replaced independently:

- capture lead criteria from a form
- scrape and filter search results for real business sites
- extract company details and generate outreach assets with AI
- send personalized cold email campaigns
- track open events with a lightweight webhook pixel
- watch inbox replies and update lead status
- notify Slack when a workflow fails

## Workflow Map

### 1. Data Collection

[workflows/data collection.json](workflows/data%20collection.json)

Starts with a form called **Lead Machine**. It collects the business type, location, lead count, and preferred email style. The workflow then queries Google search via Apify, splits organic results, filters out directories and job boards, and checks whether a site looks real before letting the lead continue.

### 2. AI Extraction

[workflows/ai extraction.json](workflows/ai%20extraction.json)

Uses Gemini-powered extraction and validation to turn a scraped lead into usable outbound data. It also performs sanity checks on email and name fields before updating the extracted-data sheet.

### 3. Outreacher

[workflows/outreacher.json](workflows/outreacher.json)

Builds the cold email, adds a tracking pixel, sends the message through Gmail, and marks the lead as sent in the outreach sheet.

### 4. Webhook Trigger

[workflows/webhook trigger.json](workflows/webhook%20trigger.json)

Handles open tracking. A webhook receives the pixel hit, calculates the time delta, marks the lead as seen, and returns a 1x1 transparent GIF so the email loads cleanly.

### 5. Reply Checker

[workflows/reply checker.json](workflows/reply%20checker.json)

Polls Gmail for inbound replies, matches them back to the lead sheet, updates the lead status to replied, and posts a Slack notification.

### 6. Error Handler

[workflows/error handler.json](workflows/error%20handler.json)

Listens for workflow errors and sends a Slack alert with the workflow name, failing node, and execution link.

## Folder Structure

```text
decoupled-agentic-pipeline/
├── README.md
├── docs/
│   └── architecture-diagram.png
└── workflows/
	├── ai extraction.json
	├── data collection.json
	├── error handler.json
	├── outreacher.json
	├── reply checker.json
	└── webhook trigger.json
```

## End-to-End Flow

1. Submit the form in **Data Collection**.
2. Search results are scraped and filtered down to real business websites.
3. Lead data is enriched and validated in **AI Extraction**.
4. **Outreacher** drafts and sends a personalized email.
5. Open events hit **Webhook Trigger** and update the status to seen.
6. Replies are detected by **Reply Checker** and marked as replied.
7. Any failure anywhere is reported by **Error Handler**.

## Notes

- The workflows are intentionally decoupled so each stage can be swapped without rewriting the whole system.
- Several nodes reference existing Google Sheets, Gmail, Slack, and Apify credentials inside n8n.
- The repository contains workflow exports only; import them into your n8n instance to run the pipeline.

## Suggested Next Steps

1. Import the workflows into n8n in the order shown above.
2. Replace the hard-coded credentials, sheet IDs, and webhook domain with your own values.
3. Test the pipeline with a small lead batch before scaling it up.
