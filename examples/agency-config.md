# Example config — marketing agency

A marketing agency's inbound is messier than a consultant's. You're getting new-business inquiries, white-label partnership pitches, support requests from existing clients who forgot the support email exists, and a wall of recruiter and SEO spam.

**Suggested classifier categories:**

- `new_business` — prospect interested in hiring the agency; budget or scope signals
- `partnership` — another agency, freelancer, or platform proposing white-label or referral arrangement
- `support_existing` — current client routing to the wrong inbox; needs a fast hand-off
- `recruiting` — candidate applying or recruiter pitching talent
- `spam` — cold SaaS outreach, link-building, generic SEO services

Update the `Classify Lead` prompt and `Parse Classification` allowed array. Route `new_business` and `partnership` to the high-priority drafter. Route `support_existing` to its own branch that pings the account-management channel in Slack with a different message (you don't want to auto-reply to a paying client with a sales draft). `recruiting` and `spam` short-circuit.

**Suggested tone tweaks for the drafter:**

> Confident and specific. Three to four sentences. Reference the channel or capability they mentioned (paid social, SEO, lifecycle, etc.). Propose a 30-minute call with a calendar link placeholder `[CALENDAR_LINK]`. Avoid promising results or naming clients.

For agencies, also worth adding a webhook output field for which `Slack channel` to ping — `#new-biz` for new business, `#partnerships` for partnership, `#account-mgmt` for client support — so notifications land where the right person sees them.
