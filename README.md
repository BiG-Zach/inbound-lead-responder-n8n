# Reply to every inbound lead in under 60 seconds — even when you're asleep.

A drop-in n8n workflow that catches inbound contact-form submissions, classifies them with an LLM, drafts a personalized reply, pings your team in Slack, and logs the whole thing — without you touching the keyboard. Built for solo agencies, consultants, advisors, contractors, real estate agents, and anyone whose pipeline starts with a contact form. If you've ever lost a deal because you saw the email three hours after the prospect already booked a competitor, this is for you.

## The problem

- **Slow replies kill deals.** Lead-response research is brutal — the contact-rate curve falls off a cliff after the first five minutes. Most solo operators reply in hours, not seconds.
- **Generic auto-responders feel like spam.** "Thanks, we'll be in touch" doesn't move anyone closer to a call. Hot leads need a real, specific reply.
- **CRM busywork eats your day.** Triaging contact-form noise, copy-pasting messages into your CRM, deciding what's urgent — it's the kind of work that should never have made it onto your calendar in the first place.

## What this template does

```
Webhook (POST /inbound-lead)
  → Normalize lead fields (with sample-data fallback)
  → Classify with OpenRouter LLM (category + urgency 1–10 + intent summary)
  → Route by category (spam/support short-circuit; hot_lead/partnership get the full treatment)
  → Draft a 3-sentence personalized reply → send via Gmail → notify Slack
  → Acknowledge the webhook with JSON
```

## What's in the box

```
inbound-lead-responder/
├── workflow.json              # The n8n workflow — import this
├── .env.example               # Env vars you'll set
├── LICENSE                    # MIT
├── README.md                  # This file
├── docs/
│   ├── setup-5-minutes.md     # Step-by-step install
│   ├── customize-the-prompt.md
│   └── swap-crms.md
└── examples/
    ├── consultant-config.md
    ├── real-estate-config.md
    └── agency-config.md
```

## Quickstart (5 minutes)

1. **Import** — In n8n, go to Workflows → Import from File → pick `workflow.json`.
2. **Create the OpenRouter credential** — Credentials → New → HTTP Header Auth. Name it `OpenRouter API`. Header name `Authorization`, value `Bearer YOUR_OPENROUTER_KEY`. Grab a key at openrouter.ai.
3. **(Optional) Connect Gmail** — Click the `Send Email Reply` node and authorize Gmail OAuth2. Skip this if you don't want auto-replies sent yet.
4. **(Optional) Set `SLACK_WEBHOOK_URL`** — Add it as an n8n environment variable or in Settings → Variables.
5. **Test it** — Click `Execute Workflow` once. The `Normalize Lead` node has a built-in sample lead, so the whole pipeline runs end-to-end without a real webhook fire. You should see a classification land in `Parse Classification` and a draft reply in `Format Reply`.

Detailed walkthrough in `docs/setup-5-minutes.md`.

## Tested with

- n8n **1.121.0+** (self-hosted and Cloud)
- OpenRouter — default model `openrouter/owl-alpha` (free tier as of writing)
- Gmail OAuth2, Slack incoming webhooks

## Security note

**This workflow requires n8n 1.121.0 or higher.** That version patches two important CVEs you should not run without:

- **CVE-2025-1217** — webhook path-traversal in older n8n builds
- **CVE-2026-21858 ("Ni8mare")** — credential exposure via the expression engine

Do not run this template on older n8n versions, particularly if your instance is internet-exposed. If you're self-hosting, run `n8n --version` before importing.

## How it works

The classifier is a single OpenRouter chat completion with a tight system prompt and `response_format: json_object`, returning `{category, urgency_score, intent_summary}`. A Code node parses the response defensively — markdown fences stripped, scores clamped, category whitelisted, fallback to `cold_lead` on any parse failure — so the workflow never dies on a malformed LLM response. The Switch node routes by category: spam and support short-circuit to the webhook ack; `hot_lead` and `partnership` get a second LLM call that drafts a 3-sentence reply referencing one specific detail from the original message. Gmail and Slack both run with `continueOnFail` so the webhook always returns 200, even if email or Slack is misconfigured.

## Customize the prompts

The two prompts live in the `Classify Lead` and `Draft High-Priority Reply` nodes. You can change categories, tone, sentence length, and language without touching any other part of the workflow. See `docs/customize-the-prompt.md`.

## Swap CRMs

The template ships CRM-free on purpose — every shop's CRM is different and pre-built integrations rot fast. Adding HubSpot, Pipedrive, or Airtable is one HTTP Request node after `Format Reply`. See `docs/swap-crms.md` for a working example.

## Get the Pro version

[**Buy the Pro pack →**](PLACEHOLDER_GUMROAD_URL)

Pro adds:

- Pre-built HubSpot, Pipedrive, and Airtable variants (drop-in, no JSON surgery)
- Companion error-handler workflow that catches failed executions and DMs you
- Sample-data demo mode (toggle a single variable to run without a real webhook source)
- 90-second Loom walkthrough — install, customize, ship
- Priority email support, 48-hour reply SLA

## License

MIT. See `LICENSE`.

## Built by

Zach Bradford / BiG-Zach — [PLACEHOLDER_X_URL](PLACEHOLDER_X_URL)

If this template saves you even one lost lead, it's paid for itself. If it doesn't work for you, email me — I'd rather refund you than have an unhappy customer on the list.
