## 2. Set up a phone number and email inbox

Inspect existing resources before creating new ones. Reuse the number, inbox,
and saved agent that fit the user's task.

### Phone number through MCP

Hail offers numbers in 120+ countries. Use live `quote_numbers` results
to check availability, voice/SMS capabilities, costs, and applicant requirements.
An offer marked `by_request=true` needs a manual support order; it cannot be
bought automatically. Coverage does not guarantee every number type or channel
in every country.

For number registration, get the requirements for the selected `quote_id` and
the actual applicant's `subject_type`. Collect the required user-supplied details,
address, and documents. Check the form before filing. Submit only with the
applicant's authorization, then follow the registration's reported next step.
Carrier approval is separate from submission and can take business days.

For US A2P 10DLC, Hail's current requirements, check, and submission interfaces
require `purpose="sms"`, `country_code="US"`, and `number_type="local"`.
Here `number_type` is a Hail interface parameter, not a choice to ask the user
to make. Supply `local` when calling those interfaces. The preview and amendment
tools do not take `number_type`; follow each discovered tool's actual schema.
Supply `number_id` when the form requires an existing sender. Preview the campaign and get the user's approval before paid submission.
Read the current fees from the requirements; do not hard-code them.
If confirmation is `code`, use the applicant's code with `confirm_verification`.
If it is `reply`, the applicant must reply to the carrier's text. Correct an
application at `needs_information` with `amend_sms_application` on the same
registration rather than creating another paid registration.

After approval, buy a current quote with `acquire_number` and the user's
confirmed setup-plus-monthly total in `confirm_total_cents`. A purchased number
does not answer automatically: create or reuse a Hail agent, then `route_number`
for calls and texts. Use `route_email_domain` to connect an inbox to the agent
for email replies.
Agents support instructions, greetings, language/voice settings, call limits,
and handover contacts. Buying a US number and registering it for US SMS are
different operations; number ownership alone does not authorize US texting.

### Phone number through REST

```bash
curl --fail-with-body -sS "https://api.hail.so/v1/numbers?limit=50" \
  -H "authorization: Bearer $HAIL_API_KEY"
```

For a new number, request `POST /v1/numbers/quotes` with `country_code` and
`capabilities` such as `["voice","sms"]`. Check the returned prices, capabilities,
and paperwork. Complete any required verification, then acquire the number with
`POST /v1/numbers`, using an unexpired `quote_id` and `country_code`.
Use the same price approval and registration checks as the MCP workflow above.
The console also supports number selection and verification.

Use the number response's `id` for routing and its E.164 phone number for `from`
on sends. A shared outbound voice line does not provide a dedicated inbound number.

### Email inbox

List the workspace's email identities first:

```bash
curl --fail-with-body -sS https://api.hail.so/v1/email-domains \
  -H "authorization: Bearer $HAIL_API_KEY"
```

For a Hail-hosted address, create a `hail_mail` identity. It uses Hail's domain
and needs no custom-domain DNS setup. The response's `domain` contains the address.

```bash
curl --fail-with-body -sS -X POST https://api.hail.so/v1/email-domains \
  -H "authorization: Bearer $HAIL_API_KEY" -H "content-type: application/json" \
  -d '{"kind":"hail_mail"}'
```

To use a custom domain, follow the domain and DNS verification workflow in
https://hail.so/docs. Sending verification and inbound mail routing have different
requirements; a verified sending domain alone does not establish an inbox.

Read incoming email through the API, or subscribe to `email.received` webhooks
so your assistant or application can process it:

```bash
curl --fail-with-body -sS "https://api.hail.so/v1/emails?direction=inbound&limit=50" \
  -H "authorization: Bearer $HAIL_API_KEY"
```

### Answer incoming calls, texts, and email

**Calls and texts:** create a saved agent with instructions and a greeting, then
route the number to it. For email, route an address to the agent.
Connect booking tools or webhooks when the task requires completing a booking.

```bash
curl --fail-with-body -sS -X POST https://api.hail.so/v1/agents \
  -H "authorization: Bearer $HAIL_API_KEY" -H "content-type: application/json" \
  -d '{"name":"Front desk","system_prompt":"Take appointment requests for Acme Dental.","first_message":"Thanks for calling. How can I help?"}'

# Set AGENT_ID and NUMBER_ID to the returned resource IDs.
# This example requires a number with both voice and SMS capabilities.
curl --fail-with-body -sS -X PATCH https://api.hail.so/v1/numbers/$NUMBER_ID \
  -H "authorization: Bearer $HAIL_API_KEY" -H "content-type: application/json" \
  -d '{"voice_agent_id":"'"$AGENT_ID"'","sms_agent_id":"'"$AGENT_ID"'"}'
```

Route only the channels the number supports. Incoming call events include
`call.received`, `call.answered`, and `call.completed`; failures have their own
outcomes. Answered inbound minutes use the same rates as outbound minutes.

**Email replies:** connect an inbox to a Hail agent so it can read the email
thread and reply. Configure it through the console, MCP, REST API, or CLI:

1. Create or update a live agent with `email_enabled: true` and email instructions.
   A greeting or phone number is not required for an email-only agent.
2. Get the inbox's `domain_id` from `list_email_domains`. A custom domain needs
   verification and inbound mail enabled.
3. Call `route_email_domain` with `domain_id`, `email_agent_id`, and
   `email_agent_scope="members"`. The REST equivalent is
   `PATCH /v1/email-domains/{domain_id}` with those agent/scope fields; the CLI
   is `hail email domain route <domain-id> --agent <agent-id>`.
4. The default scope replies to verified workspace members whose mail passes
   sender checks. Use `anyone` only after explicit approval: an outside sender
   can cause the agent to use tools and spend credits.
5. Check the reply outcome. Sender authentication, spam/automatic-mail filters,
   reply limits, agent status, and account funds can prevent a reply. Incoming
   mail is retained when replies are skipped; do not promise every email gets
   an automatic response.
