# Customize the prompts

The workflow has two LLM calls. You'll want to tune at least one of them to match your business.

## Where they live

- **Classify Lead** node — decides what kind of message just came in. System prompt sets the categories and the JSON schema. Temperature is `0.2` (deterministic).
- **Draft High-Priority Reply** node — writes the reply body. System prompt sets the tone, length, and sign-off. Temperature is `0.7` (more variation).

Open either node, click **Body → JSON**, and edit the `messages[0].content` string. That's it.

## Change the classification categories

The default categories are `hot_lead`, `cold_lead`, `support`, `spam`, `partnership`. To add new ones (say, `demo_request` and `press_inquiry`):

1. In the `Classify Lead` system prompt, append the new categories to the enum list and give each a short definition.
2. In the `Parse Classification` Code node, find the `allowed` array near the bottom and add the new category strings.
3. In the `Route by Category` Switch node, add a new output and a matching rule — or fold the new category into an existing route (e.g., treat `demo_request` like `hot_lead`).

Skip step 2 and the parser will silently fall back to `cold_lead` for any new category. Skip step 3 and the new category will route to the fallback output (which goes straight to Acknowledge).

## Change the reply tone

Edit the system prompt in `Draft High-Priority Reply`. The shipped prompt says *"warm, personalized, exactly 3 sentences, reference one specific detail, friendly but not gushing, end with a clear next step."*

Swap pieces of that for your voice. A few examples:

- **Formal / professional services:** *"Polished and concise. Four to five sentences. Reference the prospect's stated challenge. Offer a 20-minute discovery call. Sign off with [Your Name]."*
- **Casual / creator-style:** *"Conversational and direct. Use contractions. Two to three sentences max. Ask one open-ended question that gets them talking."*
- **Technical / developer audience:** *"Plain, no fluff. Acknowledge the technical specifics they mentioned. Propose a concrete next step (call, async write-up, or shared doc). Avoid sales language."*

## Adapt to your business

- **SaaS company:** Categories like `trial_signup`, `enterprise_inquiry`, `feature_request`, `bug_report`, `spam`. Reply prompt should ask about team size and use case.
- **Real estate agent:** See `examples/real-estate-config.md`.
- **Fitness coach:** Categories like `1on1_inquiry`, `group_program`, `meal_plan`, `media`, `spam`. Reply prompt should ask about current goals and timeline.

## Swap the model

The default model is `google/gemini-2.5-flash` — fast, stable, and cheap (~$0.0002–$0.0014 per lead). To swap it:

1. Open the `Classify Lead` node → **Body** → **JSON**.
2. Find the line `"model": "google/gemini-2.5-flash"` and replace with any other OpenRouter model ID.
3. Do the same in the `Draft High-Priority Reply` node.
4. Save and re-test.

**Recommended alternatives:**

- **`anthropic/claude-3.5-haiku`** — Better at nuanced classification and writing personable replies. ~$1/M input, $5/M output. Roughly 3–4x the cost of Gemini Flash but noticeably higher quality on edge cases.
- **`openai/gpt-4o-mini`** — Comparable price to Gemini Flash, slightly different prose style. Good fallback if Gemini availability ever flakes.
- **Free-tier models** — Browse the current free pool at https://openrouter.ai/collections/free-models. Useful for testing and very low-volume use. **Caveat:** free models get deprecated, rate-limited, or replaced without notice. Don't pin a business workflow to a free model unless you're prepared to re-test when it changes.
- **`openrouter/free`** — OpenRouter's auto-routing endpoint that picks from the current free pool. Trades determinism for free inference.

If you swap to a model that doesn't support `response_format: json_object` (most free models don't), remove that line from the `Classify Lead` JSON body. The `Parse Classification` Code node already handles fence-stripped JSON in plain text responses, so it'll still work — just slightly less deterministic.

## Temperature reminder

- **Lower (0.1–0.3)** — more consistent, more deterministic. Use for the classifier. You want the same message to always get the same label.
- **Higher (0.6–0.9)** — more variation, less template-y. Use for the drafter. You don't want every prospect getting the same three sentences with their name swapped.

Don't go above 1.0 unless you enjoy chaos.
