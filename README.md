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
2. **Create the OpenRouter credential** — Credentials → New → HTTP Header Auth. Name it `OpenRouter API`. Header name `Authorization`, value `Bearer YOUR_OPENROUTER_KEY`. Grab a key at openrouter.ai and add $5 credit so the default model has room to run.
3. **(Optional) Connect Gmail** — Click the `Send Email Reply` node and authorize Gmail OAuth2. Skip this if you don't want auto-replies sent yet.
4. **(Optional) Set `SLACK_WEBHOOK_URL`** — Add it as an n8n environment variable or in Settings → Variables.
5. **Test it** — Click `Execute Workflow` once. The `Normalize Lead` node has built-in sample-data fallbacks, so the whole pipeline runs end-to-end without a real webhook fire. You should see a classification land in `Parse Classification` and a draft reply in `Format Reply`.

Detailed walkthrough in `docs/setup-5-minutes.md`.

## API cost

The workflow ships with `google/gemini-2.5-flash` as the default model via OpenRouter — fast, stable, and inexpensive. Measured per-lead cost (verified on real test traffic):

- **Spam / cold lead / support** (classify only): ~$0.0002 per lead
- **Hot lead / partnership** (classify + draft reply): ~$0.0004 per lead

In practice, **$5 of OpenRouter credit covers roughly 11,500 hot-lead pipeline runs or 23,000 lower-priority leads** — most solo operators get months or years of runway per top-up.

You can swap to any other OpenRouter model — including free-tier options — in one field per node. See `docs/customize-the-prompt.md` for the model-swap instructions.

## Tested with

- n8n **1.121.0+** (self-hosted and Cloud)
- OpenRouter — default model `google/gemini-2.5-flash`
- Gmail OAuth2, Slack incoming webhooks

## Security note

**This workflow requires n8n 1.121.0 or higher.** Older versions are vulnerable to **CVE-2026-21858 ("Ni8mare")**, a critical (CVSS 10.0) unauthenticated file-read vulnerability in n8n's webhook handling, patched on November 18, 2025.

Because this template uses a webhook trigger, you should not run it on older n8n versions, especially if your instance is internet-exposed. Run `n8n --version` to verify before importing.

Reference: [GitHub Security Advisory GHSA-v4pr-fm98-w9pg](https://github.com/n8n-io/n8n/security/advisories/GHSA-v4pr-fm98-w9pg)

## How it works

The classifier is a single OpenRouter chat completion with a tight system prompt and `response_format: json_object`, returning `{category, urgency_score, intent_summary}`. A Code node parses the response defensively — markdown fences stripped, scores clamped, category whitelisted, fallback to `cold_lead` on any parse failure — so the workflow never dies on a malformed LLM response. The Switch node routes by category: spam and support short-circuit to the webhook ack; `hot_lead` and `partnership` get a second LLM call that drafts a 3-sentence reply referencing one specific detail from the original message. Gmail and Slack both run with `continueOnFail` so the webhook always returns 200, even if email or Slack is misconfigured.

## Customize the prompts

The two prompts live in the `Classify Lead` and `Draft High-Priority Reply` nodes. You can change categories, tone, sentence length, language — and the model itself — without touching any other part of the workflow. See `docs/customize-the-prompt.md`.

## Swap CRMs

The template ships CRM-free on purpose — every shop's CRM is different and pre-built integrations rot fast. Adding HubSpot, Pipedrive, or Airtable is one HTTP Request node after `Format Reply`. See `docs/swap-crms.md` for a working example.

## Pricing & versions

This template ships in three tiers on Gumroad:

- **Lite — $19.** Just the `workflow.json`, README, and LICENSE. The exact same files shipped on GitHub. Best for: "I just want the workflow."
- **Pro — $49.** Everything in Lite, plus the `docs/` folder (setup, customize, swap-CRMs), the `examples/` folder (consultant, real estate, agency configs), `.env.example`, a 2-minute video walkthrough, and priority email support with a 48-hour reply SLA.
- **Bundle — $129.** Everything in Pro, plus my next n8n template — **Inbound Email Triage** — shipping within 30 days. Bundle buyers get early access before the public launch.

[**Buy on Gumroad →**](https://bigzachai.gumroad.com/l/sdxzdj)

## License

MIT. See `LICENSE`.

## Built by

Zach Bradford / BiG-Zach

If this template saves you even one lost lead, it's paid for itself. If it doesn't work for you, email me — I'd rather refund you than have an unhappy customer on the list.
