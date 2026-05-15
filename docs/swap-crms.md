# Swap CRMs

The template ships **without a CRM node by default.** That's deliberate — every shop's CRM is different, pre-built integrations break, and you almost certainly want to map fields your own way. Adding your CRM is a single HTTP Request node.

## Where to add it

Insert the new node **after `Format Reply` and before `Send Email Reply`**. At that point in the workflow every useful field is available on `$json`: `name`, `email`, `company`, `source`, `message`, `category`, `urgency_score`, `intent_summary`, `reply_subject`, `reply_body`.

If you'd rather have the CRM call happen in parallel with the email (so a slow CRM API doesn't delay the prospect's reply), add a second branch off `Format Reply` instead of inserting in-line.

## Example: HubSpot

Create or update a contact via HubSpot's CRM v3 API. You'll need a private app token — see the HubSpot docs for how to create one — stored as an n8n credential named `HubSpot Private App` (HTTP Header Auth, header `Authorization`, value `Bearer YOUR_TOKEN`).

```http
POST https://api.hubapi.com/crm/v3/objects/contacts
Content-Type: application/json
Authorization: Bearer YOUR_PRIVATE_APP_TOKEN

{
  "properties": {
    "email": "{{ $json.email }}",
    "firstname": "{{ $json.name.split(' ')[0] }}",
    "lastname": "{{ $json.name.split(' ').slice(1).join(' ') }}",
    "company": "{{ $json.company }}",
    "hs_lead_status": "{{ $json.category === 'hot_lead' ? 'NEW' : 'OPEN_DEAL' }}",
    "lead_source": "{{ $json.source }}",
    "inbound_intent_summary": "{{ $json.intent_summary }}",
    "inbound_urgency_score": {{ $json.urgency_score }}
  }
}
```

If the contact already exists, HubSpot returns a 409 — handle that by switching to the **upsert by email** endpoint, or by chaining a search-then-update flow.

## Pipedrive

Same shape, different URL — `POST https://your-domain.pipedrive.com/api/v2/persons?api_token=YOUR_TOKEN`. Map fields to `name`, `email`, `org_name`, and a custom field for the intent summary.

## Airtable

Use the Airtable node that ships with n8n. Point it at your "Leads" table and map the columns. Cleanest of the three since you don't have to think about field IDs or API versions.

## Pro version

The Pro pack on Gumroad includes pre-wired HubSpot, Pipedrive, and Airtable variants — drop-in JSON files with the CRM node already inserted, field mappings ready, and the upsert logic working. Skip this doc and import the variant if you'd rather not hand-build it.
