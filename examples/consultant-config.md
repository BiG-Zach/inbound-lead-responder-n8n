# Example config — solo consultant

A solo strategy or management consultant has a different inbound mix than a SaaS company. Most messages are some flavor of "can we talk about working together" — but the *kind* of conversation matters, and the wrong reply tone will lose a referral.

**Suggested classifier categories:**

- `discovery_call` — prospect wants to explore an engagement, has at least vague budget or timeline
- `referral` — someone is sending you a name; reply needs to acknowledge the referrer
- `partnership` — agency, platform, or fellow consultant proposing collaboration
- `low_fit` — interesting but clearly not a fit (wrong industry, wrong size, no budget signal)
- `spam` — cold outbound, link-building pitches, SEO services

Update the `Classify Lead` system prompt with these categories and short definitions. Update the `Parse Classification` Code node's `allowed` array to match. In `Route by Category`, send `discovery_call`, `referral`, and `partnership` to the high-priority branch; let `low_fit` and `spam` short-circuit to the webhook ack.

**Suggested tone tweaks for the drafter:**

> Polished and considered. Four sentences. Reference one specific business detail from their message. Propose a concrete next step — a 25-minute introductory call with two suggested time windows. Avoid jargon and avoid mentioning rates.

Lower the temperature to `0.5` if you want more consistency across replies.
