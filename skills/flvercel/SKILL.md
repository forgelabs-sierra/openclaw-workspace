---
name: flvercel
description: Deploy apps and manage Vercel projects, domains, and environment variables for forgelabs projects. All operations go through the host API — no credentials needed in sandbox.
metadata: {"clawdbot":{"emoji":"","requires":{"bins":["flvercel"]}}}
---

# flvercel — Vercel Deployment & Management for Sandbox

Manages Vercel projects, deployments, domains, and environment variables via the host clitools API.
No Vercel credentials exist in the sandbox — all auth is handled on the host.

## When to use

- Deploying applications to Vercel
- Checking deployment status and history
- Managing custom domains on Vercel projects
- Setting or viewing environment variables
- Promoting preview deployments to production

## Important: Security

- **No project deletion** — must be done by the operator in the Vercel dashboard
- **No deployment deletion** — operator only
- Domain add/remove/update and env add/remove are allowed (needed for functional workflows)
- Domain update can pin domains to git branches or set up redirects
- **Always ask the user** before deploying to production or modifying env vars

## Project commands

```bash
flvercel projects list                          # list all projects
flvercel projects inspect my-project            # project details + latest deployment

# Create project (import GitHub repo) — idempotent (no-op if project already exists)
flvercel projects create my-app --repo forgelabs-sierra/my-app                    # default framework: nextjs
flvercel projects create my-api --repo forgelabs-sierra/my-api --framework auto   # auto-detect framework
flvercel projects create my-spa --repo forgelabs-sierra/my-spa --framework vite   # specify framework

# Common --framework values:
#   nextjs (default), vite, remix, sveltekit, astro, nuxtjs, gatsby, hugo
#   fastapi, flask, django
#   create-react-app, vuejs, angular, ember
#   auto (let Vercel detect)
```

## Deployment commands

```bash
flvercel deployments list                       # list recent deployments
flvercel deployments list --project my-app      # filter by project
flvercel deployments list --target production   # production only
flvercel deployments inspect dpl_abc123         # deployment details

flvercel deploy --project my-app --ref feature/xyz  # preview deploy (no --prod = preview)
flvercel deploy --project my-app --ref main --prod  # production deploy (--prod required)
flvercel deploy --project my-app --prod              # production deploy (latest commit)

flvercel promote dpl_abc123                     # promote preview to production
```

## Domain commands

```bash
flvercel domains list --project my-app                      # list domains
flvercel domains add myapp.forgelabs.nz --project my-app    # add domain
flvercel domains add myapp.forgelabs.nz --project my-app --branch feature/cms  # add with branch pin
flvercel domains remove myapp.forgelabs.nz --project my-app # remove domain
flvercel domains verify myapp.forgelabs.nz --project my-app # verify domain

# Update domain configuration (branch pinning, redirects)
flvercel domains update myapp.forgelabs.nz --project my-app --branch feature/cms   # pin to branch
flvercel domains update myapp.forgelabs.nz --project my-app --branch ""            # unpin (serve production)
flvercel domains update myapp.forgelabs.nz --project my-app --redirect other.com --redirect-code 301  # redirect
```

## Environment variable commands

```bash
flvercel env list --project my-app              # list env vars (values hidden)
flvercel env list --project my-app --decrypt    # list with decrypted values

flvercel env add --project my-app DATABASE_URL "postgres://..." --target production
flvercel env add --project my-app API_KEY "sk-..." --target production,preview

flvercel env remove --project my-app OLD_KEY    # remove from all targets
flvercel env remove --project my-app OLD_KEY --target production  # specific target
```

## Typical workflows

### Deploy a project (git-based)

```bash
# 1. Push code to GitHub
git add . && git commit -m "update"
flgh push

# 2a. Deploy to production (--prod flag required)
flvercel deploy --project my-app --prod --ref main

# 2b. Deploy to preview (omit --prod — deploys as preview)
flvercel deploy --project my-app --ref feature/xyz
```

**Important:** Without `--prod`, all deploys are preview. Use `--prod` only when intentionally deploying to production. Normal workflow: push to branch → Vercel auto-deploys preview. Only use `flvercel deploy --ref` to force a redeploy without pushing a new commit.

### Add custom domain with forgelabs.nz DNS

```bash
# 1. Add CNAME in DNS (use fldns skill)
fldns create myapp CNAME cname.vercel-dns.com

# 2. Add domain in Vercel
flvercel domains add myapp.forgelabs.nz --project my-app

# 3. Verify (may take a moment for DNS propagation)
flvercel domains verify myapp.forgelabs.nz --project my-app
```

Use `.test` suffix for non-production:
- Test: `fldns create myapp.test CNAME cname.vercel-dns.com` -> `myapp.test.forgelabs.nz`
- Production: `fldns create myapp CNAME cname.vercel-dns.com` -> `myapp.forgelabs.nz`

### Set environment variables

```bash
flvercel env add --project my-app DATABASE_URL "postgres://user:pass@host/db" --target production
flvercel env add --project my-app NEXT_PUBLIC_API_URL "https://api.example.com" --target production,preview
```

### Check deployment status

```bash
flvercel deployments list --project my-app --limit 5
flvercel deployments inspect dpl_abc123
```

### Pin domain to a feature branch (preview environment)

```bash
# Pin sierrawins.test.forgelabs.nz to feature/cms branch
flvercel domains update sierrawins.test.forgelabs.nz --project forgelabs-web --branch feature/cms

# Later, unpin to go back to serving production
flvercel domains update sierrawins.test.forgelabs.nz --project forgelabs-web --branch ""
```

### Create a new project from GitHub repo

```bash
# 1. Create GitHub repo (if it doesn't exist)
flgh repo create my-app --desc "My new app"

# 2. Push code to GitHub
git add . && git commit -m "initial commit"
flgh push -u --branch main

# 3. Create Vercel project linked to GitHub repo (auto-deploys on push)
flvercel projects create my-app --repo forgelabs-sierra/my-app

# 4. Add custom domain (optional)
fldns create myapp CNAME cname.vercel-dns.com
flvercel domains add myapp.forgelabs.nz --project my-app
flvercel domains verify myapp.forgelabs.nz --project my-app

# 5. Set env vars (optional)
flvercel env add --project my-app API_KEY "sk-..." --target production
```

**Idempotent:** Running `flvercel projects create` on an existing project is a no-op — it reports the project exists and does nothing.

## Limitations

- **No project deletion** — must be done in Vercel dashboard
- **No deployment delete** — must be done by operator
- **Git-based deploy only** — projects must be connected to GitHub
- **File-based deploy not supported** — use `flgh push` + `flvercel deploy` workflow
