# 📧 Email Summarizer — WhatsApp Edition

An n8n workflow that monitors your Gmail inbox, summarizes new emails using an LLM, and sends the summary to your WhatsApp number via the Evolution API (or any compatible WhatsApp HTTP gateway).

---

## How It Works

```
Gmail Trigger → Extract Fields → Summarize (LLM) → Send to WhatsApp
```

1. **Gmail Trigger** — polls your inbox every minute for new emails.
2. **Code (JS)** — extracts the sender, subject, and snippet from the raw Gmail payload.
3. **HTTP Request (LLM)** — sends the email content to an OpenRouter-compatible endpoint and gets a 3–5 bullet point summary back.
4. **HTTP Request (WhatsApp)** — formats the summary and sends it to your WhatsApp number via an HTTP API.

---

## Prerequisites

- n8n (self-hosted or cloud)
- A Gmail account connected via OAuth2
- An OpenRouter API key (or any OpenAI-compatible endpoint)
- A WhatsApp HTTP gateway (e.g. [Evolution API](https://github.com/EvolutionAPI/evolution-api), WPPConnect, etc.)

---

## Setup

### 1. Gmail OAuth2 Credential

In n8n, go to **Credentials → New → Gmail OAuth2** and connect your Google account. Assign it to the **Gmail Trigger** node.

### 2. LLM API (HTTP Request node)

In the first **HTTP Request** node, fill in:

| Field | Value |
|---|---|
| `url` | Your OpenRouter or OpenAI-compatible endpoint (e.g. `https://openrouter.ai/api/v1/chat/completions`) |
| `Authorization` header | `Bearer YOUR_OPENROUTER_API_KEY` |
| `model` | `openai/gpt-4o-mini` (or swap for any model you prefer) |

### 3. WhatsApp API (HTTP Request1 node)

In the second **HTTP Request** node, fill in:

| Field | Value |
|---|---|
| `url` | Your Evolution API endpoint (e.g. `https://your-instance.com/message/sendText/YOUR_INSTANCE`) |
| Auth header | Your Evolution API key (replace the `xxx` header name with the correct one for your gateway, e.g. `apikey`) |
| `number` | Your WhatsApp number in international format, no `+` (e.g. `254712345678`) |

---

## Message Format

The WhatsApp message will look like this:

```
📧 *New Email*

*From:* sender@example.com
*Subject:* Re: Project Update

*Summary:*
• The client approved the proposal
• Next meeting is Thursday at 3pm
• Action items assigned to the team
```

---

## Notes

- The Gmail Trigger uses the **snippet** field (first ~100 chars of the email body), not the full body. If you need full body content, switch to the Gmail node's `Get Message` action with `Raw` format.
- Polling every minute means up to 60 seconds of delay before you get a WhatsApp notification. Adjust the poll interval under the trigger's **Poll Times** settings.
- The LLM prompt is in the first HTTP Request node's JSON body — edit it there if you want a different summary style or language.
- To avoid getting notified about emails you don't care about, add a **Filter** node after the Gmail Trigger and filter by sender, subject keywords, or labels.

---

