# Lead Capture → CRM Sync

An n8n automation that captures leads from a webhook, validates and enriches the
email address, logs the lead to a Google Sheets CRM, and alerts the sales team on
Slack in real time — with a dedicated error path if enrichment ever fails.

## What it does

1. **Webhook** — receives a new lead as a POST request (`name`, `email`, `company`).
2. **HTTP Request (Disify)** — validates the email: checks format, domain, DNS/MX
   records, and flags disposable/temporary addresses.
3. **Google Sheets** — appends the lead, plus the enrichment results and a
   timestamp, as a new row in a "Leads CRM" sheet.
4. **Slack** — posts a formatted message to `#newleads` with the lead's details
   and validation status.
5. **Error handling** — if the enrichment API call fails, the workflow doesn't
   silently drop the lead. It routes to a separate Slack message warning that
   enrichment failed for that email, so nothing falls through the cracks.

## Workflow diagram

```
Webhook → HTTP Request (Disify) ──success──→ Google Sheets → Slack (new lead)
                                └──error────→ Slack (enrichment failed alert)
```

## Stack

| Step | Tool |
|---|---|
| Trigger | n8n Webhook node |
| Email validation | [Disify](https://disify.com) (free, no-auth API) |
| Data store | Google Sheets |
| Notifications | Slack |

## Setup

1. Import `lead-capture-crm-sync.json` into your n8n instance (n8n Cloud or
   self-hosted): **Workflows → Import from File**.
2. Connect your **Google Sheets** account and create a sheet named `Leads CRM`
   with these column headers in row 1:
   `Name | Email | Company | Email Valid | Disposable | Timestamp`
3. Connect your **Slack** account and create a `#newleads` channel.
4. Open the Webhook node and copy the **Production URL**.
5. Publish/activate the workflow.
6. Send a POST request to the production URL with a JSON body like:
   ```json
   { "name": "Jane Doe", "email": "jane@example.com", "company": "Acme Inc" }
   ```
   The lead will appear in Google Sheets and a Slack notification will fire
   within a couple of seconds.

## Example payload

```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "company": "Acme Inc"
}
```

## Demo

*[Add a link to your screen recording here once you've made one.]*

## Notes

- The Disify API is free and requires no API key, which keeps this workflow
  zero-cost to run and easy for anyone to reproduce.
- Swap the Google Sheets node for HubSpot, Airtable, or any other CRM node —
  the rest of the workflow (validation, Slack alerts, error handling) stays
  the same.
- Built as part of a portfolio project demonstrating n8n workflow design,
  API integration, and basic production-readiness (error handling, real
  notifications) rather than a bare demo.
