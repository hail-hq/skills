## 5. Handle limits and errors

- **401:** check the API key. **403:** inspect scopes, compliance, or account
  restrictions. Do not bypass restrictions or recipient suppressions.
- **402:** ask the user to add credits through the claim link or console.
- **409:** inspect the conflict. Resource names, signup, and idempotency have
  different recovery steps; do not treat every conflict as a retry signal.
- **422:** inspect `detail` for an invalid field or missing sender/verification.
  Fix the cause before another attempt; do not invent IDs or attestations.
- **429:** respect API `Retry-After` and back off. Agent workspaces have channel
  and recipient limits. Signup has a separate daily limit.
- **5xx or a timeout after a send:** the outcome may be unknown. Check saved IDs
  and history, or replay the identical request with its saved idempotency key.
  Do not blindly submit another send.
