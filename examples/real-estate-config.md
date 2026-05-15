# Example config — real estate agent

Real estate inbound is high-volume and time-sensitive — a buyer who messaged at 9pm and didn't hear back by 9:15pm is already messaging the next agent on Zillow. Speed matters more here than almost any other vertical.

**Suggested classifier categories:**

- `buyer_lead` — looking to purchase, may mention price range, neighborhood, or timeline
- `seller_lead` — wants a listing presentation or CMA; mentions their property
- `showing_request` — asking about a specific MLS listing; usually time-sensitive
- `vendor_pitch` — photographer, stager, mortgage broker, lead-gen service pitching you
- `spam` — generic SEO offers, off-topic outreach

Update the `Classify Lead` system prompt and `Parse Classification` allowed array. Route `buyer_lead`, `seller_lead`, and `showing_request` to the high-priority branch. `vendor_pitch` and `spam` short-circuit.

**Suggested tone tweaks for the drafter:**

> Friendly, fast, and concrete. Three short sentences. Reference the specific neighborhood, price range, or listing they mentioned. Offer two times in the next 48 hours for a call or showing. Sign off with [Your Name].

Bump urgency weighting in the classifier prompt so `showing_request` returns urgency 9–10 by default — those need an under-10-minute reply.
