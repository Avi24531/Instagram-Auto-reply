# NexFlowz — Instagram Reel Auto-Reply Automation

An n8n workflow that automatically listens for incoming Instagram DMs/comments via the Meta Graph API webhook, generates a context-aware reply using an AI agent (OpenAI or Google Gemini), sends the reply back to the user, and logs every interaction to a Google Sheet for tracking and analytics.

## Overview

This workflow handles the full lifecycle of an automated Instagram reply:

1. **Webhook verification** — Responds to Meta's webhook handshake (`hub.challenge`) so Instagram can confirm the endpoint.
2. **Incoming message handling** — Captures incoming DM/comment payloads from the Instagram webhook.
3. **AI-generated response** — Routes the message text to an AI Agent node, which can use either OpenAI (`gpt-4o-mini`) or Google Gemini as the underlying language model.
4. **Response formatting** — Cleans and extracts the AI's reply text, handling both plain text and structured JSON outputs.
5. **Reply delivery** — Sends the generated reply back to the user via the Instagram Graph API.
6. **Logging** — Appends each interaction to a Google Sheet for record-keeping and reporting.

## Workflow Diagram

```
Webhook ──┬──> If (verify token check) ──> Respond to Webhook
          │
          └──> AI Agent ──> Edit Fields ──> Code in JavaScript ──┬──> HTTP Request (send reply) ──┐
                  │                                              │                                 ├──> Append row in sheet
            ┌─────┴─────┐                                        └─────────────────────────────────┘
     OpenAI Chat Model   Google Gemini Chat Model
```

## Prerequisites

- An active [n8n](https://n8n.io/) instance (self-hosted or cloud)
- A Meta Developer App with Instagram messaging permissions configured
- An OpenAI API key and/or Google Gemini API key
- A Google account with Sheets API access for logging

## Setup Instructions

### 1. Import the workflow

In your n8n instance, go to **Workflows → Import from File** and select [`workflows/auto-reply-instagram.json`](auto-reply-instagram.json).

### 2. Configure credentials

This workflow requires the following credentials to be set up in n8n (**Settings → Credentials**):

| Credential | Used by | Notes |
|---|---|---|
| OpenAI API | OpenAI Chat Model | Generate at [platform.openai.com](https://platform.openai.com/api-keys) |
| Google Gemini API | Google Gemini Chat Model | Generate via [Google AI Studio](https://aistudio.google.com/) |
| Google Sheets OAuth2 | Append row in sheet | Connect your Google account with Sheets access |

### 3. Set environment-specific values

Copy [`.env.example`](.env.example) to `.env` and fill in your actual values for reference, then update the corresponding fields directly inside the n8n nodes:

- **Webhook node** — note the generated webhook URL; you'll register this with Meta.
- **If node** — replace `YOUR_VERIFY_TOKEN` with the verify token you set in your Meta App's webhook configuration.
- **HTTP Request node** — replace `YOUR_ACCESS_TOKEN` with your Instagram Graph API access token.
- **Append row in sheet node** — select your target Google Sheet document and tab.

### 4. Register the webhook with Meta

In your Meta Developer App dashboard, set the webhook callback URL to your n8n webhook URL (the `instagram-incomming` path) and use the same verify token configured in step 3.

### 5. Activate the workflow

Toggle the workflow to **Active** in n8n. Incoming Instagram messages will now be processed automatically.

## Node Reference

- **Webhook** — Entry point for all incoming requests from Meta (both GET verification and POST events).
- **If** — Validates the `hub.verify_token` and `hub.mode` during Meta's webhook handshake.
- **Respond to Webhook** — Returns the `hub.challenge` value to complete verification.
- **AI Agent** — LangChain agent node that processes the incoming message text and generates a reply, with OpenAI as primary and Gemini as fallback (or vice versa, depending on configuration).
- **Edit Fields** — Normalizes the AI Agent's output into a consistent field.
- **Code in JavaScript** — Parses the AI output (handling both plain text and nested JSON) and extracts the sender ID and clean message text.
- **HTTP Request** — Sends the formatted reply back to the user via the Instagram Graph API.
- **Append row in sheet** — Logs the sender ID and message text to Google Sheets for analytics/auditing.

## Security Notes

- This repository does **not** contain any real credentials, tokens, or instance-specific identifiers. All sensitive values are replaced with placeholders (e.g. `YOUR_ACCESS_TOKEN`, `YOUR_VERIFY_TOKEN`).
- Never commit your `.env` file or n8n credential exports to version control.
- Rotate your Instagram access token and verify token if they were ever exposed.

## Roadmap

- [ ] Add support for Instagram comment replies in addition to DMs
- [ ] Add rate limiting / spam detection before AI processing
- [ ] Add a fallback static response if both AI models fail
- [ ] Add sentiment tagging in the Google Sheets log

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.
