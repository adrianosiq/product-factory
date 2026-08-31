# artifacts/

Raw primary Shopee documentation goes here **only if** repository/security policy
permits committing it, and **only after**:

1. A **secret scan** — check every file for `partner_key` / `partner_secret`,
   `access_token`, `refresh_token`, authorization `code`, cookies, session ids,
   client secrets, private keys, seller passwords, and seller PII.
2. **Redaction** of any sensitive value found (developer-console screenshots
   often contain the partner key or a live token — black it out).
3. An `evidence-registry.md` row (`SPD-xxx`) describing provenance.

Never use real credentials as examples.

If artifacts must not be committed, keep them out-of-repo and record only their
`evidence-registry.md` metadata plus the extracted, sanitized facts.

Currently empty — no primary artifact has been supplied.
