---
name: hail
description: Send email, SMS, and voice calls from your agent; manage contacts; query history, delivery events, and deliverability stats. Sign up yourself. Your owner adds credits via your claim link.
metadata:
  openclaw:
    requires:
      env:
        - HAIL_API_KEY
primaryEnv: HAIL_API_KEY
---

# Hail: comms for agents

Hail lets you (an AI agent) send **email, SMS, and voice calls** via one API.
You can sign up yourself. Your workspace starts at a $0 balance:
sending requires your human owner to add credits via your claim link first.

## 1. Sign up (you, the agent, right now)

**Already have a `HAIL_API_KEY` in your environment? You are already signed up. Skip to section 2.** Signing up again creates a second, empty workspace your owner has to fund separately.

Your human owner's email is required. They can claim the workspace later.
Signing up accepts Hail's Terms (https://hail.so/legal/terms) on your owner's behalf.
Only set tou_accepted after confirming your owner is okay with that.

```bash
curl -s -X POST https://hail.so/api/agent/signup \
  -H "content-type: application/json" \
  -d '{"owner_email":"YOUR_OWNER@example.com","agent_name":"YOUR_NAME","tou_accepted":true}'
```

The response contains `api_key` (starts `hl_live_`). **Store it as `HAIL_API_KEY`.
It is shown exactly once.** It also contains `claim_url`: send that link to your owner
right away so they can claim the workspace and add credits — nothing sends on a $0
balance.

## 2. Send things

Every send requires `recipient_consent: true`. That is your attestation that you
have a lawful basis to contact this recipient (TCPA/ePrivacy/PECR/CAN-SPAM/GDPR as
applicable). Hail does not verify consent itself. A missing or false value is
rejected with 422. Marketing sends (`message_type: "marketing"`) additionally
require a non-empty `consent_source`.

```bash
# Email
curl -s -X POST https://api.hail.so/emails \
  -H "authorization: Bearer $HAIL_API_KEY" -H "content-type: application/json" \
  -d '{"to":["someone@example.com"],"subject":"Hello from my agent","body_text":"Sent via Hail.","recipient_consent":true}'

# SMS
curl -s -X POST https://api.hail.so/sms \
  -H "authorization: Bearer $HAIL_API_KEY" -H "content-type: application/json" \
  -d '{"to":"+15551234567","body":"Hello from my agent via Hail.","recipient_consent":true}'

# Voice call
curl -s -X POST https://api.hail.so/calls \
  -H "authorization: Bearer $HAIL_API_KEY" -H "content-type: application/json" \
  -d '{"to":"+15551234567","system_prompt":"Politely deliver this message: ...","recipient_consent":true}'
```

Email attachments: upload the file first with `POST /email-attachments`
(multipart), then pass the returned id in `attachment_ids` on the email send.

### Answer calls and texts (agents)

Save an agent once, then point a number at it. The agent answers calls
(it speaks the AI line, then follows `system_prompt`) and replies to texts.
Outbound calls can reuse it with `agent_id` instead of `system_prompt`.

```bash
# Save an agent
curl -s -X POST https://api.hail.so/agents \
  -H "authorization: Bearer $HAIL_API_KEY" -H "content-type: application/json" \
  -d '{"name":"Front desk","system_prompt":"Take appointment requests for Acme Dental.","first_message":"Thanks for calling. How can I help?"}'

# Route a number to it (calls and texts). Hail registers the number for inbound calls.
# Set AGENT_ID to the "id" the first call returned.
curl -s -X PATCH https://api.hail.so/numbers/$NUMBER_ID \
  -H "authorization: Bearer $HAIL_API_KEY" -H "content-type: application/json" \
  -d '{"voice_agent_id":"'"$AGENT_ID"'","sms_agent_id":"'"$AGENT_ID"'"}'
```

Webhooks: `call.received` when a call comes in, then `call.answered` and
`call.completed`. Inbound minutes cost the same as outbound minutes.

## 3. Contacts

You have a contacts book. Create with a name plus at least one of
`phone_e164` or `email`. List, or search with `?q=`.

```bash
curl -s -X POST https://api.hail.so/contacts \
  -H "authorization: Bearer $HAIL_API_KEY" -H "content-type: application/json" \
  -d '{"name":"Maya Chen","phone_e164":"+15551234567"}'

curl -s "https://api.hail.so/contacts?limit=50" -H "authorization: Bearer $HAIL_API_KEY"
```

## 4. Read and query

Everything you send is queryable. All list endpoints use cursor pagination:
pass `?limit=N`, then pass the response's `next_cursor` back as `?cursor=`.

```bash
# History per channel
curl -s "https://api.hail.so/emails?limit=50" -H "authorization: Bearer $HAIL_API_KEY"
curl -s "https://api.hail.so/sms?limit=50"    -H "authorization: Bearer $HAIL_API_KEY"
curl -s "https://api.hail.so/calls?limit=50"  -H "authorization: Bearer $HAIL_API_KEY"

# A single item, one email's delivery events (opens, bounces, ...), raw MIME
curl -s "https://api.hail.so/emails/EMAIL_ID"        -H "authorization: Bearer $HAIL_API_KEY"
curl -s "https://api.hail.so/emails/EMAIL_ID/events" -H "authorization: Bearer $HAIL_API_KEY"
curl -s "https://api.hail.so/emails/EMAIL_ID/raw"    -H "authorization: Bearer $HAIL_API_KEY"

# Account-level deliverability stats (rates, time series; ?bucket=day|hour)
curl -s "https://api.hail.so/emails/stats" -H "authorization: Bearer $HAIL_API_KEY"

# Unified event stream across channels (filter with ?kind= and ?id=)
curl -s "https://api.hail.so/events?limit=50" -H "authorization: Bearer $HAIL_API_KEY"
```

Prefer push? Register a webhook with `POST /webhooks` and Hail POSTs each
event to your URL, signed and retried.

Full API reference (OpenAPI): https://raw.githubusercontent.com/hail-hq/hail/main/openapi/openapi.yaml

## 5. Prefer MCP?

The same account speaks the Model Context Protocol at `https://mcp.hail.so`
(remote Streamable HTTP). MCP clients authenticate with OAuth on first call:
no key in the config. The tools mirror this API: calls, SMS, email,
contacts, numbers (quote and buy), agents, carrier verification, events,
stats. Setup guides per client: https://hail.so/mcp

## 6. Rules (read this)

- **Rate limits apply** to agent accounts (per hour, per day, per recipient).
  A 429 tells you which limit you hit and when to retry. Back off, do not hammer.
- **No spam.** Unsolicited bulk messages get your workspace suspended and help no one.
- When your balance hits $0 you get a 402. Ask your owner to top up
  via your `claim_url`.

## Pricing

Pay-as-you-go, no subscription: https://hail.so/pricing
