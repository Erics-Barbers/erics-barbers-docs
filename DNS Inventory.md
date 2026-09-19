# DNS Inventory

## Purpose

This document records the public DNS routing for `erics-barbers-luton.co.uk` and explains the purpose of each custom record. Squarespace manages the authoritative DNS zone.

This is a sanitized operational inventory, not an export of the registrar dashboard. Verification tokens, DKIM public-key material, account details, and credentials are intentionally omitted. Confirm the live authoritative DNS before making or recovering infrastructure changes.

Last reconciled from the Squarespace custom-record view on 19 September 2026.

## Web And API Routing

| Host | Type | Public target | Environment | Purpose |
| --- | --- | --- | --- | --- |
| `@` | `A` | `216.198.79.1` | Production | Routes the apex customer domain to Vercel. |
| `api` | `CNAME` | `gayhg526.up.railway.app` | Production | Routes the production API hostname to the Railway API service. |
| `staff` | `CNAME` | `d82e70ea76e22037.vercel-dns-017.com` | Production | Routes the production staff hostname to Vercel. |
| `test` | `CNAME` | `d82e70ea76e22037.vercel-dns-017.com` | Test | Routes the customer test hostname to the Vercel test deployment. |
| `test-api` | `CNAME` | `nubeop7h.up.railway.app` | Test | Routes the test API hostname to the Railway test API service. |
| `test-staff` | `CNAME` | `d82e70ea76e22037.vercel-dns-017.com` | Test | Routes the staff test hostname to the Vercel test deployment. |

The resulting service hostnames are:

- customer production: `erics-barbers-luton.co.uk`
- staff production: `staff.erics-barbers-luton.co.uk`
- API production: `api.erics-barbers-luton.co.uk`
- customer test: `test.erics-barbers-luton.co.uk`
- staff test: `test-staff.erics-barbers-luton.co.uk`
- API test: `test-api.erics-barbers-luton.co.uk`

## Transactional Email

| Host | Type | Priority | Public value | Purpose |
| --- | --- | ---: | --- | --- |
| `send.mail` | `MX` | `10` | `feedback-smtp.eu-west-1.amazonses.com` | Provides the custom MAIL FROM return path used by Resend through Amazon SES. |
| `_dmarc` | `TXT` | - | `v=DMARC1; p=none;` | Publishes the current DMARC monitoring policy. |
| `send.mail` | `TXT` | - | `v=spf1 include:amazonses.com ~all` | Authorizes Amazon SES to send mail for the custom MAIL FROM subdomain. |
| `resend._domainkey.mail` | `TXT` | - | Omitted | Publishes the DKIM public key used to authenticate transactional email. |

The DMARC policy currently monitors mail without requesting quarantine or rejection. Any move to an enforcing policy should follow delivery verification and an explicit operational decision.

## Provider Verification

| Host | Type | Value recorded here | Purpose |
| --- | --- | --- | --- |
| `_railway-verify.api` | `TXT` | Omitted | Proves control of the production API hostname to Railway. |
| `_railway-verify.test-api` | `TXT` | Omitted | Proves control of the test API hostname to Railway. |

Verification values are published in DNS and therefore externally observable, but retaining their complete values in source control adds no recovery value. Retrieve current required values from Railway when revalidating or replacing a domain.

## Ownership And Change Safety

- Squarespace owns authoritative DNS record management.
- Vercel terminates and routes the customer and staff web hostnames.
- Railway terminates and routes the production and test API hostnames.
- Resend, backed by Amazon SES DNS records, provides transactional email.
- Production and test hostnames must remain connected to their matching environments.
- Do not point `test`, `test-staff`, or `test-api` at production services.
- Do not remove provider-verification or mail-authentication records until the associated provider confirms they are no longer required.
- Record the purpose and environment here whenever a custom record is added, changed, or removed.

## Information That Must Not Be Recorded

Do not add registrar credentials, domain-transfer codes, API keys, mail credentials, private keys, billing details, or account-recovery information to this inventory. Screenshots of the DNS dashboard should be redacted and kept out of the repository.
