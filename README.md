# make-ai-lead-qualification-agent
Automated sales lead intake and classification pipeline built in Make.com using Google Gemini AI, JSON parsing, Google Sheets logging, and conditional Gmail routing with English and Urdu support. Classic first automation task. 

# AI Lead Qualification Agent (Make.com + Google Gemini)

An automated sales lead intake and triage pipeline built with **Make.com** and **Google Gemini AI** for a custom software development agency. Incoming client inquiries arrive via a custom webhook, are classified into actionable lead tiers (`hot`, `warm`, `cold`) using structured JSON outputs, logged to **Google Sheets**, and conditionally routed to **Gmail** for instant escalation of high-intent or urgent leads.

## Architecture & Workflow

```text
Custom Webhook ──▶ Google Gemini AI (Flash) ──▶ Google Sheets (Agent Log) ──▶ Router ──▶ Gmail (Hot Leads Only)
```

1. **Custom Webhook:** Receives incoming JSON payloads containing `name` and `message` (supports UTF-8 for both English and Urdu inquiries).
2. **Google Gemini AI (`gemini-3.5-flash-lite`):** Evaluates the message using a structured system prompt and outputs raw JSON (`category`, `reason`, `reply`).
3. **Google Sheets (`Agent Log`):** Appends every lead with its classification, reasoning, and tailored draft reply.
4. **Router & Filter:** Evaluates `category == "hot"` and dispatches immediate email alerts via Gmail while filtering out `warm` and `cold` inquiries.

## Lead Classification Criteria

| Category | Definition | Automated Action |
| :--- | :--- | :--- |
| **`hot`** | High intent, urgent production bug, or ready to sign/kick off a project immediately | Logged to Google Sheets + Instant Gmail alert |
| **`warm`** | Interested in web/app development, asking general pricing or timeline questions without urgency | Logged to Google Sheets only |
| **`cold`** | Low intent, just browsing, irrelevant inquiries, or unsolicited spam | Logged to Google Sheets only |

## Setup & Replication

1. **Create the Google Sheet:**
   * Create a spreadsheet with a sheet tab named `Agent Log` and headers in Row 1 (`A1:E1`):
     `Name` | `Message` | `Category` | `Reason` | `Reply`
2. **Import the Blueprint:**
   * Open **Make.com** → **Scenarios** → **Create a new scenario**.
   * Click the **More (`...`)** menu at the bottom toolbar → **Import Blueprint**, and upload `blueprint.json`.
3. **Configure Connections & IDs:**
   * **Webhook:** Generate a new Custom Webhook URL.
   * **Google Gemini AI:** Connect your Google AI Studio API key and select a Flash model. Set **Max Output Tokens** to `1000+` under advanced settings to prevent JSON truncation.
   * **Google Sheets:** Reconnect your Google account and select your `Agent Log` spreadsheet.
   * **Gmail:** Connect your Gmail account and set the recipient address for urgent `hot` lead alerts.
4. **Activate:**
   * Save the scenario (`Ctrl+S`) and toggle **Scheduling** to **ON**.

## System Prompt

```text
You are an AI sales intake assistant for a custom software development house in Islamabad.
Read the incoming client message and classify the lead.
Return ONLY raw JSON, no code fences, no extra text, in this exact shape:
{"category": "...", "reason": "one short sentence", "reply": "a short friendly reply"}

category must be one of: hot, warm, cold.
- hot = High intent, urgent production bug, or ready to start a software project immediately.
- warm = Interested in web or app development, asking general questions or pricing, no immediate urgency.
- cold = Low intent, just browsing, irrelevant message, or spam.

Keep the reply professional, welcoming, and representative of a software engineering agency. Do not use dashes in the reply.
```

## Sample Test Payloads (`curl`)

Replace `YOUR_WEBHOOK_URL` with your active Make.com webhook endpoint before testing.

### English Hot Lead (Urgent Production Issue)
```bash
curl -X POST "YOUR_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"name": "Tariq Mahmood", "message": "Our production database stopped syncing after an update and orders are failing. Need an emergency engineer on call today."}'
```

### English Warm Lead (Pricing Inquiry)
```bash
curl -X POST "YOUR_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"name": "Ayesha Malik", "message": "Hi, how much do you generally charge for a custom WordPress website and how many weeks does it take?"}'
```

### English Cold Lead (Spam Filter Test)
```bash
curl -X POST "YOUR_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"name": "PromoBot99", "message": "CONGRATS!! Click here to claim free crypto rewards and SEO backlinks for your domain today!!!"}'
```

### Urdu Hot Lead (UTF-8 Localization Test)
*(On Windows Command Prompt, run `chcp 65001` first to enable UTF-8 encoding.)*
```bash
curl -X POST "YOUR_WEBHOOK_URL" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{"name": "فہد محمود", "message": "ہمارا بجٹ منظور ہو چکا ہے اور ہم اسی ہفتے اپنی ای کامرس ایپ کا کام شروع کرنا چاہتے ہیں۔ آج میٹنگ رکھ لیں؟"}'
```

## Sample AI Output

```json
{
  "category": "hot",
  "reason": "The client has an emergency production failure requiring immediate technical intervention today.",
  "reply": "Hello Tariq, we are dispatching an emergency engineer right now to investigate and resolve your database sync issue."
}
```
