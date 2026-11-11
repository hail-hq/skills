## 3. Send email, texts, and calls

Send only when the user authorized the task and the recipient can lawfully be
contacted. `recipient_consent: true` records your attestation; Hail does not
obtain consent for you. A missing or false value is rejected with 422.
Marketing sends also need `message_type: "marketing"` and a non-empty `consent_source`.

Check the channel's setup before sending:

- **Email:** omit `from` to use the resolved default sender, or select a verified
  identity explicitly. Multiple eligible senders require an explicit `from`.
  Custom domains need DNS verification. Open and click tracking requires `body_html`.
- **SMS:** the US, Canada, and other destinations require an active workspace-owned
  SMS number. US traffic also needs approved SMS registration. Some destinations
  support an alphanumeric sender ID without a dedicated number. SMS never borrows
  a shared voice line.
- **Calls:** omit `from` to use an active workspace voice number or a shared line.
  Use `agent_id` to reuse a saved agent, or `system_prompt` for this call's task.
- **Credits:** the workspace must be funded. Credits do not replace sender or
  carrier requirements.

Examples need curl 7.76 or later. Replace recipients and placeholders with values
for the actual task. Set and save a unique request ID for each intended send,
such as `EMAIL_REQUEST_ID`. The examples stop if that variable is unset.

```bash
# Email
curl --fail-with-body -sS -X POST https://api.hail.so/v1/emails \
  -H "Idempotency-Key: ${EMAIL_REQUEST_ID:?Set and save a unique ID for this send}" \
  -H "authorization: Bearer $HAIL_API_KEY" -H "content-type: application/json" \
  -d '{"to":["recipient@example.com"],"subject":"Appointment details","body_text":"Here are the details you requested.","recipient_consent":true}'

# Text: this US example requires the number and registration setup in setup.md.
curl --fail-with-body -sS -X POST https://api.hail.so/v1/sms \
  -H "Idempotency-Key: ${SMS_REQUEST_ID:?Set and save a unique ID for this send}" \
  -H "authorization: Bearer $HAIL_API_KEY" -H "content-type: application/json" \
  -d '{"to":"+15551234567","body":"Your appointment request has been received.","recipient_consent":true}'

# Call
curl --fail-with-body -sS -X POST https://api.hail.so/v1/calls \
  -H "Idempotency-Key: ${CALL_REQUEST_ID:?Set and save a unique ID for this send}" \
  -H "authorization: Bearer $HAIL_API_KEY" -H "content-type: application/json" \
  -d '{"to":"+15551234567","system_prompt":"Confirm the appointment details provided by the user. Do not invent missing details.","recipient_consent":true}'
```

Inspect each response before continuing. Save the returned resource `id` and
check its status or events: an accepted request does not prove delivery.

If a response is lost, replay the identical body with its saved `Idempotency-Key`.
A new key can create a second send. Do not enable automatic send retries without
idempotency protection. Failures may also be cached; after resolving a confirmed
rejection, use a new key for the new attempt. Inspect 409 errors before retrying:
the original request may still be running, or the key may have different data.

For attachments, upload a file with `POST /v1/email-attachments` using multipart
field `file`, then include the returned `id` in the email's `attachment_ids`.
