---
name: webdev
description: Web development workflow for Forge Labs — Next.js apps on Vercel with GitHub, Google OAuth, custom domains, and visual QA via browser screenshots. Covers the full lifecycle from scaffold to production.
metadata: {"clawdbot":{"emoji":"","requires":{"bins":["flgh","flvercel","fldns"]}}}
---

# webdev — Web Development Workflow for Forge Labs

End-to-end workflow for building and deploying Next.js web applications on Vercel with Forge Labs infrastructure. Covers project creation, deployment, auth setup, DNS, visual QA, and common gotchas.

## When to use

- Building a new Next.js web application
- Deploying to Vercel with a custom forgelabs.nz subdomain
- Setting up Google OAuth for admin areas
- Wiring up GitHub CMS (content editing via API commits)
- Doing visual QA with browser screenshots
- Troubleshooting Vercel deployment or DNS issues

## Tech stack (Forge Labs standard)

- **Framework:** Next.js 15 (App Router, TypeScript, `src/` directory)
- **CSS:** Tailwind CSS v3 (NOT v4 — see known issues below)
- **Components:** shadcn/ui v2
- **Icons:** Lucide React
- **Auth:** Auth.js v5 (NextAuth) with Google OAuth
- **Deployment:** Vercel (git-based)
- **DNS:** Cloudflare via fldns
- **GitHub:** forgelabs-sierra account via flgh

---

## Full project lifecycle

### Step 1: Scaffold the project

```bash
# Create Next.js project (in workspace)
cd /workspace
npx create-next-app@latest my-project --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
cd my-project

# Install shadcn
npx shadcn@latest init
# When prompted: style=new-york, base-color=neutral, css-variables=yes

# Add commonly needed shadcn components
npx shadcn@latest add button card badge input textarea sheet separator avatar

# Install other deps as needed
npm install lucide-react
```

**IMPORTANT — use Tailwind CSS v3, NOT v4.** See known issues section below.

### Step 2: Build and test locally

```bash
npm run build    # verify build passes
npm run dev      # local dev server on port 3000 (not accessible from outside sandbox)
```

Build must pass before pushing to GitHub. Fix all TypeScript and build errors first.

### Step 3: Push to GitHub

```bash
# Initialize git and make first commit
git init
git add .
git commit -m "Initial scaffold"

# Create the GitHub repo (ask user for name and visibility first)
flgh repo create my-project --desc "Description here"

# Add remote and push
flgh remote add forgelabs-sierra/my-project
flgh push -u --branch main
```

### Step 4: Import project in Vercel (MANUAL — cannot automate)

**This is the one step that requires the operator (Cameron) in the Vercel dashboard.** `flvercel` cannot create projects.

Three-step process:
1. Go to **vercel.com** → click **"Add New..."** → **"Project"**
2. **Import Git Repository** → select **forgelabs-sierra/my-project** from the list
3. Click **Deploy** — accept all defaults (Vercel auto-detects Next.js)

The first deployment will auto-run on import. It may fail if env vars aren't set yet — that's fine, we'll redeploy after setting them.

**Vercel also requires a GitHub PAT** to access forgelabs-sierra repos. If not already connected:
- The Vercel GitHub integration handles this via OAuth when you import
- Or: create a fine-grained PAT on github.com/settings/tokens scoped to the specific repo

### Step 5: Wire up DNS

```bash
# Create CNAME pointing to Vercel (use .test subdomain for non-production)
fldns create myapp.test CNAME cname.vercel-dns.com

# CRITICAL: Disable Cloudflare proxy — Vercel needs direct DNS
fldns update myapp.test --no-proxy
```

**Cloudflare proxy MUST be disabled for Vercel custom domains.** If proxy is on, you get `ERR_SSL_VERSION_OR_CIPHER_MISMATCH` because Cloudflare's SSL conflicts with Vercel's. This is the most common deployment issue.

```bash
# Add the domain to the Vercel project
flvercel domains add myapp.test.forgelabs.nz --project my-project

# Wait ~30 seconds for DNS propagation, then verify
flvercel domains verify myapp.test.forgelabs.nz --project my-project
```

### Step 6: Set environment variables

```bash
# Set env vars on the Vercel project
flvercel env add --project my-project AUTH_SECRET "$(openssl rand -hex 32)" --target production
flvercel env add --project my-project NEXTAUTH_URL "https://myapp.test.forgelabs.nz" --target production
# ... add others as needed

# Verify they're set
flvercel env list --project my-project
```

**Important:** After setting env vars, trigger a redeploy for them to take effect:

```bash
flvercel deploy --project my-project --ref main --prod
```

### Step 7: Visual QA with browser screenshots

See the "Browser screenshots for visual QA" section below.

---

## Google OAuth setup (Auth.js v5)

### What you need

| Env var | Value | Where to get it |
|---------|-------|-----------------|
| `AUTH_GOOGLE_ID` | OAuth 2.0 Client ID | Google Cloud Console |
| `AUTH_GOOGLE_SECRET` | OAuth 2.0 Client Secret | Google Cloud Console |
| `AUTH_SECRET` | Random 32+ char string | `openssl rand -hex 32` |
| `NEXTAUTH_URL` | Production URL (e.g., `https://myapp.forgelabs.nz`) | Your deployment URL |

### GCP OAuth setup — step by step

1. Go to **console.cloud.google.com**
2. Select the **forgelabs.nz** project (or create one if needed)
3. Left menu → **APIs & Services** → **Credentials**
4. Click **+ Create Credentials** → **OAuth 2.0 Client ID**
5. If prompted to configure the **OAuth consent screen** first:
   - **User type:** Internal (restricts to forgelabs.nz org automatically — simplest and most secure option)
   - **App name:** (e.g., "Forge Labs Website")
   - **Support email:** cameron@forgelabs.nz
   - Save and continue through the rest (scopes: leave defaults, no test users needed for Internal)
6. Back on Credentials → **+ Create Credentials** → **OAuth 2.0 Client ID**
   - **Application type:** Web application
   - **Name:** (e.g., "Forge Labs Website")
   - **Authorized JavaScript origins:**
     - `https://myapp.forgelabs.nz` (production URL)
     - `http://localhost:3000` (local dev)
   - **Authorized redirect URIs:**
     - `https://myapp.forgelabs.nz/api/auth/callback/google` (production)
     - `http://localhost:3000/api/auth/callback/google` (local dev)
   - Click **Create**
7. Copy the **Client ID** → `AUTH_GOOGLE_ID` env var
8. Copy the **Client Secret** → `AUTH_GOOGLE_SECRET` env var

### Auth.js code pattern

```typescript
// src/lib/auth.ts
import NextAuth from "next-auth"
import Google from "next-auth/providers/google"

export const { handlers, auth, signIn, signOut } = NextAuth({
  providers: [
    Google({
      clientId: process.env.AUTH_GOOGLE_ID!,
      clientSecret: process.env.AUTH_GOOGLE_SECRET!,
    }),
  ],
  callbacks: {
    async signIn({ account, profile }) {
      // Restrict to forgelabs.nz Google Workspace domain
      if (account?.provider === "google") {
        return (
          profile?.email_verified === true &&
          (profile?.email?.endsWith("@forgelabs.nz") ?? false)
        )
      }
      return false
    },
  },
})
```

```typescript
// src/middleware.ts
export { auth as middleware } from "@/lib/auth"
export const config = {
  matcher: ["/admin/:path*", "/api/content/:path*"],
}
```

```typescript
// src/app/api/auth/[...nextauth]/route.ts
import { handlers } from "@/lib/auth"
export const { GET, POST } = handlers
```

**Session strategy:** JWT (stateless, no database needed). Auth.js encrypts the session into a cookie using `AUTH_SECRET`.

**Domain restriction:** The `signIn` callback checks `profile.email.endsWith("@forgelabs.nz")`. Non-forgelabs users complete the OAuth flow but get redirected to `/login?error=AccessDenied`.

---

## GitHub PAT for CMS (content editing via API)

If the app has a CMS that commits content changes to GitHub:

### Fine-grained PAT (recommended)

Create at **github.com/settings/tokens** under the forgelabs-sierra account:

- **Token type:** Fine-grained personal access token
- **Repository access:** Only select repositories → select the specific repo (e.g., `forgelabs-sierra/forgelabs-web`)
- **Permissions:**
  - **Contents:** Read and write (to commit content changes)
- Set as `GITHUB_TOKEN` env var on Vercel

### Content API pattern (Octokit)

```typescript
import { Octokit } from "@octokit/rest"

// GET — fetch content from repo
const octokit = new Octokit({ auth: process.env.GITHUB_TOKEN })
const { data } = await octokit.repos.getContent({
  owner: process.env.GITHUB_OWNER!,
  repo: process.env.GITHUB_REPO!,
  path: "content/home.md",
})
// data.content is base64-encoded, data.sha needed for updates

// PUT — commit updated content
await octokit.repos.createOrUpdateFileContents({
  owner, repo,
  path: "content/home.md",
  message: "Update homepage content",
  content: Buffer.from(newContent).toString("base64"),
  sha: currentSha,  // prevents conflicts — must match current file SHA
})
```

---

## Browser screenshots for visual QA

Use the browser tool to take screenshots of deployed sites for visual inspection.

### Two wait periods required

1. **Browser cold start (first use after idle):** The browser container (headless Chromium) spins up Chrome on first use. The first connection attempt may timeout. **If the first attempt fails, retry once** — the second attempt succeeds because Chrome is now running.

2. **Page load wait (every screenshot):** After navigating to a URL, **wait 10-30 seconds** before taking a screenshot. The page needs time for:
   - JavaScript rendering (React hydration, dynamic content)
   - Background images to load (Unsplash hero images, etc.)
   - Fonts to load (FOUT → rendered text)
   - CSS animations to complete

   **If you screenshot too fast, you get a blank or partially loaded image.**

### Screenshot workflow

```
1. Open the browser tool → navigate to URL
2. Wait at least 10 seconds (longer for image-heavy pages)
3. Take screenshot
4. The screenshot saves to /home/openclaw/.openclaw/media/browser/<uuid>.png
   (accessible from sandbox at /data/browser-screenshots/)
5. Use the image tool to analyse the screenshot
```

### Image model status

**Note:** The image model (vision) is currently being configured. If you get "Image model connection issue" errors, the image tool cannot analyse screenshots yet. Workaround:
- You can still take screenshots and describe what you see based on the page structure
- The screenshot files are viewable by the operator (Cameron) on the host
- A fix is in progress — `agents.defaults.imageModel` needs to route through LiteLLM

When the image model is working, screenshots from the gateway path (`/home/openclaw/.openclaw/media/browser/...`) should be directly analysable without copying.

---

## Using background agents for builds

For large build tasks (scaffolding + building + deploying), use a background sub-agent to avoid blocking the main conversation:

- Spawn the agent with the full build instructions
- Check on progress periodically
- The agent handles: scaffold → build → fix errors → push to GitHub → deploy
- Expect 15-30 minutes for a full scaffold-to-deploy cycle
- Common blockers: build errors, missing deps, Tailwind issues

**Monitor progress:** Check the agent's output to see where it's at. Key milestones:
1. Project scaffolded
2. Build passes (`npm run build` succeeds)
3. GitHub repo created and code pushed
4. Vercel deployment triggered
5. DNS and domain configured
6. Env vars set and redeployed

---

## Known issues and gotchas

### Tailwind CSS v4 — DO NOT USE in sandbox

**Issue:** `@tailwindcss/oxide` native binaries fail to install on Linux ARM64 (Docker/Podman). This is a well-known ongoing issue with Tailwind v4's native binary approach.

**Symptoms:**
- `npm install` or `npm run build` fails with native binding errors
- `@tailwindcss/oxide` compilation errors
- Build hangs or crashes

**Fix:** Use **Tailwind CSS v3** instead. When scaffolding with `create-next-app`, it may install v4 by default. Downgrade:

```bash
npm install tailwindcss@3 postcss autoprefixer
```

And ensure `tailwind.config.ts` exists (v3 uses JS config, not CSS-first).

### Cloudflare proxy conflicts with Vercel SSL

**Issue:** Cloudflare proxy (orange cloud) intercepts HTTPS traffic and provides its own SSL certificate. Vercel also provisions SSL via Let's Encrypt. When both try to handle SSL, you get `ERR_SSL_VERSION_OR_CIPHER_MISMATCH`.

**Fix:** Always disable Cloudflare proxy for Vercel-hosted domains:

```bash
fldns create myapp.test CNAME cname.vercel-dns.com
fldns update myapp.test --no-proxy    # CRITICAL
```

**How to diagnose:** If you see SSL errors on a newly deployed Vercel site:
```bash
fldns list | grep myapp    # Check if proxy is ON
fldns update myapp.test --no-proxy
# Wait 30-60 seconds for propagation
```

### Vercel env vars require redeploy

Setting env vars via `flvercel env add` doesn't affect running deployments. You must trigger a new deployment for the vars to take effect:

```bash
flvercel deploy --project my-project --ref main --prod
```

### Contact form — FormData not JSON

If the site's contact form submits to a Google Apps Script, send as **FormData** (not JSON). Apps Script `doPost(e)` reads `e.parameter`, which expects URL-encoded form data:

```typescript
// WRONG — Apps Script won't parse this
fetch(url, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name, email, message }),
})

// CORRECT — Apps Script reads e.parameter from FormData
const formData = new FormData()
formData.append("firstName", firstName)
formData.append("lastName", lastName)
formData.append("email", email)
formData.append("message", message)
fetch(url, { method: "POST", body: formData })
```

### Subdomain naming convention

- **Non-production:** `<name>.test.forgelabs.nz` (e.g., `sierrawins.test.forgelabs.nz`)
- **Production:** `<name>.forgelabs.nz` (e.g., `forgelabs.nz`)

Always ask the user whether test or production before creating DNS records. Default to `.test`.

---

## Visual design lessons learned

### Hero sections with background images

- Use `next/image` with `fill` and `priority` for hero backgrounds
- Apply dark overlay (`bg-black/50` or `bg-black/65`) so white text is readable
- Unsplash provides optimized URLs: `?w=1920&q=80&auto=format`
- Configure `next.config.ts` for Unsplash domain:
  ```ts
  images: {
    remotePatterns: [
      { protocol: 'https', hostname: 'images.unsplash.com' },
    ],
  },
  ```

### Making a site look premium

Lessons from iterating on the Forge Labs site:

- **Flat cards look cheap.** Add: rounded-2xl corners, solid coloured icon backgrounds, hover effects (lift + shadow), subtle top border accents
- **Use real typography scale.** Section headings should be much larger than body text. Add eyebrow labels above headings (small caps, primary colour)
- **Dark sections create drama.** Hero and CTA benefit from near-black backgrounds with blue tint gradients. Footer should be dark to close the page
- **Whitespace matters.** Generous padding between sections (`py-20` to `py-32`). Cards need internal padding too
- **Icons need presence.** Small inline icons get lost. Put them in coloured circles (`bg-primary text-white rounded-full p-3`)
- **Hover interactions.** Cards: `hover:-translate-y-1 hover:shadow-lg`. Buttons: `hover:scale-[1.02]`. Service cards: background fill on hover

---

## Reference: Forge Labs website env vars

For the current forgelabs-web project (`sierrawins.test.forgelabs.nz`):

| Var | Value |
|-----|-------|
| `AUTH_GOOGLE_ID` | `<redacted>` |
| `AUTH_GOOGLE_SECRET` | `<redacted>` |
| `AUTH_SECRET` | (generated — stored in Vercel) |
| `GITHUB_OWNER` | `forgelabs-sierra` |
| `GITHUB_REPO` | `forgelabs-web` |
| `NEXTAUTH_URL` | `https://sierrawins.test.forgelabs.nz` |

GCP OAuth config:
- **App type:** Web application
- **Name:** Forge Labs Website
- **Consent screen:** Internal (forgelabs.nz org only)
- **JS origins:** `https://sierrawins.test.forgelabs.nz` + `http://localhost:3000`
- **Redirect URIs:** `https://sierrawins.test.forgelabs.nz/api/auth/callback/google` + `http://localhost:3000/api/auth/callback/google`

---

## Quick reference — deployment checklist

```
[ ] Project scaffolded (Next.js + Tailwind v3 + shadcn)
[ ] Build passes locally (npm run build)
[ ] GitHub repo created (flgh repo create)
[ ] Code pushed (flgh push -u --branch main)
[ ] Vercel project imported (MANUAL — operator does this in dashboard)
[ ] DNS CNAME created (fldns create)
[ ] Cloudflare proxy DISABLED (fldns update --no-proxy)
[ ] Domain added to Vercel (flvercel domains add)
[ ] Domain verified (flvercel domains verify)
[ ] Env vars set on Vercel (flvercel env add)
[ ] Redeployed with env vars (flvercel deploy --prod)
[ ] Site loads on custom domain (check with browser tool)
[ ] OAuth login works (if applicable)
[ ] Contact form submits (if applicable)
```
