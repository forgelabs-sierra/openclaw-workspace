---
name: fldns
description: Create, list, and update A/CNAME DNS records for forgelabs.nz subdomains. Use after deploying a Vercel app to point a subdomain at it.
metadata: {"clawdbot":{"emoji":"🌐","requires":{"bins":["fldns"]}}}
---

# fldns — DNS Management for forgelabs.nz

## What this does

Manages DNS records for the `forgelabs.nz` domain via the `fldns` CLI.
You can create, list, and update A and CNAME records for subdomains.

## When to use

- After deploying a Vercel app, to point a subdomain at it
- To check what subdomains already exist
- To create A records pointing subdomains to specific IP addresses
- To toggle Cloudflare proxy on/off for existing records

## Available commands

### List records

```bash
fldns list                  # All records
fldns list --type A         # Only A records
fldns list --type CNAME     # Only CNAME records
fldns list --name app       # Filter by subdomain
```

### Create a CNAME record (for Vercel apps)

```bash
fldns create <subdomain> CNAME cname.vercel-dns.com
```

Example: `fldns create myapp CNAME cname.vercel-dns.com`
Result: `myapp.forgelabs.nz` points to Vercel.

Multi-level subdomains are supported:
`fldns create dashboard.test CNAME cname.vercel-dns.com`
Result: `dashboard.test.forgelabs.nz`

### Create an A record

```bash
fldns create <subdomain> A <ip-address>
```

Example: `fldns create api A 76.76.21.21`

### Update proxy status

```bash
fldns update <subdomain> --no-proxy    # Switch to DNS-only (grey cloud)
fldns update <subdomain> --proxy       # Switch to Cloudflare-proxied (orange cloud)
```

Example: `fldns update dashboard.test --no-proxy`
Result: `dashboard.test.forgelabs.nz` switches to DNS-only mode.

If multiple records exist for the same subdomain (e.g., both A and CNAME), specify the type:
```bash
fldns update dashboard.test --no-proxy --type CNAME
```

### Options

- `--no-proxy` — Disable Cloudflare proxying (DNS-only mode)
- `--proxy` — Enable Cloudflare proxying (orange cloud)
- `--comment "text"` — Add a comment to the record (create only)

## Subdomain convention

- Use `<name>.test.forgelabs.nz` for non-production (dev, staging, experiments)
- Use `<name>.forgelabs.nz` for production

**Always ask the user whether this is test or production before creating a DNS record.** Default to `.test` unless the user explicitly says production.

Examples:
- `fldns create dashboard.test CNAME cname.vercel-dns.com` — test/dev
- `fldns create dashboard CNAME cname.vercel-dns.com` — production

## Typical Vercel workflow

1. Deploy app with Vercel skill
2. Add custom domain (DNS-only — **Vercel requires `--no-proxy`** for SSL to work):
   `fldns create myapp.test CNAME cname.vercel-dns.com --no-proxy`
3. Add domain in Vercel: `flvercel domains add myapp.test.forgelabs.nz --project my-project`
4. Verify domain: `flvercel domains verify myapp.test.forgelabs.nz --project my-project`
5. Check DNS: `fldns list --name myapp.test`
6. When ready for production: `fldns create myapp CNAME cname.vercel-dns.com --no-proxy`

**Important:** Vercel custom domains must use DNS-only mode (`--no-proxy`). Cloudflare proxying (orange cloud) breaks Vercel's SSL certificate provisioning. If a record was created with proxying on, fix it with:
```bash
fldns update myapp.test --no-proxy
```

## Limitations

- **No delete** — you cannot delete existing records. Ask the operator if needed.
- **Update limited to proxy toggle** — you can switch proxy on/off but not change content or type
- **A and CNAME only** — no MX, TXT, AAAA, or other record types
- **Subdomains of forgelabs.nz only** — no other domains, no apex record
