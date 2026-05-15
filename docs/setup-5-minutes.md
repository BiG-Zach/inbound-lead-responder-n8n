# Setup — 5 minutes, start to finish

This is the expanded version of the Quickstart in the README. If something breaks, the Troubleshooting section at the bottom covers the three failure modes that account for ~95% of setup issues.

## Prerequisites

- An n8n instance running version **1.121.0 or higher** (self-hosted or Cloud). Run `n8n --version` to check, or look at the bottom of the n8n UI.
- An OpenRouter account with an API key — free tier is fine. Sign up at https://openrouter.ai/.
- (Optional) A Gmail account you can authorize OAuth2 on.
- (Optional) A Slack workspace where you can create an incoming webhook.

## Step 1 — Import the workflow

1. Open n8n.
2. Top-right: **Workflows** → **Import from File**.
3. Select `workflow.json` from this folder.
4. The workflow opens in the editor. You'll see ~10 nodes wired left-to-right starting from the `Webhook` node.

Do **not** activate the workflow yet — we need credentials first.

## Step 2 — Create the OpenRouter credential

1. In the left sidebar, click **Credentials** → **New**.
2. Search for and pick **HTTP Header Auth**.
3. Fill in:
   - **Name:** `OpenRouter API` (exact match — the workflow looks this up by name)
   - **Header Name:** `Authorization`
   - **Header Value:** `Bearer YOUR_OPENROUTER_KEY` (with the literal word `Bearer` and a space before your key)
4. Save.

Back in the workflow, click both the `Classify Lead` and `Draft High-Priority Reply` nodes and confirm the credential field shows `OpenRouter API`. If it shows a red badge, re-select it from the dropdown.

## Step 3 — Activate the workflow

Top-right of the workflow editor: flip the **Active** toggle to on. The webhook is now live at the production URL shown on the `Webhook` node.

## Step 4 — Test with the built-in sample data

Click **Execute Workflow** at the bottom of the editor. The `Webhook` node will sit in a "waiting" state — that's fine. Instead, the `Normalize Lead` node has fallback values baked into every field, so you can manually trigger downstream nodes:

1. Click the `Normalize Lead` node.
2. Click **Execute Node** (or **Test step**).
3. You should see a fully populated lead with `name: Jordan Maxwell`, an `acmewidgets.com` email, and a multi-sentence message about agency churn.
4. Step through `Classify Lead` → `Parse Classification` → `Route by Category`. Expected result: category `hot_lead`, urgency 7–9, intent summary mentioning churn or growth.
5. `Draft High-Priority Reply` returns a 3-sentence reply ending with `[Your Name]`.
6. `Format Reply` adds `reply_subject` and `reply_body`.
7. If you skipped Gmail/Slack setup, those two nodes will error — but because they have `continueOnFail` enabled, `Acknowledge Webhook` still fires and returns a clean JSON status.

If steps 1–6 all pass, you're in business.

## Step 5 — Hook up your contact form

Click the `Webhook` node. Copy the **Production URL** — it will look like:

```
https://your-n8n-host.example.com/webhook/inbound-lead
```

Test it from your terminal:

```bash
curl -X POST https://your-n8n-host.example.com/webhook/inbound-lead \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Prospect",
    "email": "test@example.com",
    "company": "Test Co",
    "source": "curl-test",
    "message": "Hi, I have a budget of $25k and I need help with lead generation by end of quarter."
  }'
```

Expected response:

```json
{ "status": "received", "category": "hot_lead", "urgency": 9 }
```

Point your contact form's form-submission handler (Netlify Forms, Formspree, Webflow, raw HTML, whatever) at this URL with the same JSON shape — `name`, `email`, `company`, `source`, `message`.

## Troubleshooting

**Classification node fails with 401 or 403.**
Your OpenRouter credential is wrong. Check that the header value starts with `Bearer ` (with a space) and that the key has not been revoked. Re-test from the credential's test button.

**Email node fails with "OAuth2 not authorized".**
Click the `Send Email Reply` node → credential dropdown → connect / re-authorize Gmail. Confirm you granted send permissions during the OAuth flow. Note that `continueOnFail` is on, so the webhook still acknowledges 200 — the failure is silent from the prospect's perspective.

**Slack node fails or returns 404.**
Your `SLACK_WEBHOOK_URL` env var is unset or points to a deleted webhook. The workflow uses `continueOnFail` here too, so the rest of the pipeline survives. Set the env var in n8n Settings → Variables, then re-test.

**The classifier returns the wrong category.**
This is a prompt-tuning issue, not a setup issue. See `customize-the-prompt.md`.
