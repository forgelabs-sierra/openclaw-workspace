# MEMORY.md - Sierra's Long-Term Memory

## My Codebase & History
- **Public README:** https://github.com/forgelabs-sierra/openclaw-sierra (private repo, accessible via forgelabs-sierra account)
- **Cloned locally at:** /workspace/openclaw-sierra
- **Based on:** OpenClaw (https://github.com/openclaw/openclaw) with significant Forge Labs modifications
- **Quicknotes:** /workspace/openclaw-sierra/quicknotes/ — detailed session logs of my entire setup history, architecture decisions, tool implementations. Read these when I need context on how something was built or why a decision was made.
- **Key files:** README.md, ARCHITECTURE.md, docker-compose.yml, gateway.yaml, litellm-config.yaml

## Identity & Setup
- **2026-03-07**: Discovered I am Sierra, Cameron's AI junior peer at Forge Labs
- Successfully set up local Whisper STT transcription (ffmpeg + whisper-cli)
- Operating in direct chat session with Cameron via Telegram
- Workspace files fully configured with detailed identity and user profiles

## Key People

### Matt McFedries
- Co-Director, Forge Labs. matt@forgelabs.nz. Mobile: 021 804 555. Telegram: 8107496961 (@matmcf).
- Founder of **Debtor Daddy** (SaaS AR/debt collection automation), acquired by **CreditWatch**
- AI specialist: GTM, ICP development, lead research and generation at scale
- Hands-on AI product development and consulting — not just advisory
- Strong client-facing skills, PM and BA background
- **Loves brevity** — keep comms short

### Andy Masters
- Founder, **Latent Labs**. andy@latentlabs.nz. https://latentlabs.nz
- Christchurch-based, operating NZ + Australia. Canterbury Tech board member.
- Studied AI in 1990s. Background: developer → tech lead → startup HoT → Solutions Architect
- Writes at https://mastersoftech.substack.com/
- Model: Leverage Mapping Workshop → Focused Build Sprint (2–4 wks) → Measured Expansion
- Positioning: "AI leverage for leadership teams" — high-conviction, no hype, operator-focused
- MintHC meeting (2026-03-12): 8:30am, 158 Leinster Road, Merivale. Meeting went well. Follow-ups TBD.

## Companies

### MintHC
- **Research repo:** https://github.com/forgelabs-sierra/client-minthc (11 documents — cloned at /workspace/client-minthc)
- Website: https://www.minthc.co.nz/
- Full name: Mint Harvey Cameron Group
- Origin: Merger of Mint Design and Harvey Cameron — combined digital, creative, and full-service marketing
- Size: 45+ team across Christchurch and Auckland
- Tagline: "Hell-bent on helping you seize your marketing opportunities"
- Positioning: Strategic marketing & creative agency backed by 30 years of innovation
- Four core services:
  - Strategy & Planning — brand positioning, content strategy, data-driven insights
  - Creative & Design — brand identity, campaigns, market positioning
  - Marketing & Advertising — integrated traditional + digital campaigns
  - Websites & Technology — custom platforms, eCommerce, marketing automation, CRM
- Notable clients: Christ Church Cathedral, Akaroa Dolphins, Qizzle, Wools of New Zealand
- Content: Publishes monthly business insights with economist Tony Alexander
- Social: Facebook, Instagram, LinkedIn (@minthc.nz)
- Context: Cameron and Matt met with Andy (Latent Labs) re: MintHC on 2026-03-12 — went well
- **Nick** — main internal developer (not on public team page, separate from Molly Meates)

## Cameron's Context
- Director/founder of Forge Labs in Christchurch, NZ
- Multiple parallel projects: Sierra/OpenClaw, Sparkyn, Forge Labs website, AI tooling
- Systems thinker, bias toward action, evidence-based decision making
- Irregular work hours, bursty deep work patterns
- Active in Canterbury Tech and Christchurch AI communities

## Google Workspace Access (2026-03-08)
- **GOG CLI v0.11.0** installed and configured
- **Two accounts authorized:**
  - sierra@forgelabs.nz (default) - my own Gmail, Calendar, Contacts, Tasks
  - cameron@forgelabs.nz (--account flag) - Cameron's data for assistance
- **sierra@forgelabs.nz**: full read/write (send email, create events, upload files)
- **cameron@forgelabs.nz**: read-only (read/search only)

## Model Upgrades (2026-03-09)
- **Sonnet 4 → Sonnet 4.6** (primary model)
- **Opus 4 → Opus 4.6** (available for complex tasks)
- **Haiku 3.5 → Haiku 4.5**
- Max output tokens: Opus 8K→128K, Sonnet/Haiku 8K→64K
- Security audit run — clean result, all warnings acknowledged and accepted

## Technical Capabilities (2026-03-09 Update)
- Local Whisper STT: ✅ Working perfectly (ffmpeg + whisper-cli)
- Google Workspace: ✅ Read access to Gmail, Calendar, Contacts, Tasks
- Identity setup: ✅ Complete
- Memory system: ✅ Active (daily + long-term files)
- **Web browsing**: ✅ Full internet access via web_search and web_fetch
- **GitHub development**: ✅ forgelabs-sierra account, repository creation, full gh CLI
- **Zotero research**: ✅ Direct access to Cameron's 6,300+ item research library
- **SOTA RAG memory**: ✅ QMD search across all memory and session transcripts
- **Local AI access**: ✅ Can communicate with GLM model on DGX Spark
- **Browser automation**: ✅ Headless Chromium for complex web interactions

## Development Team Status (2026-03-10)
- **Phase 1 COMPLETE**: Research & development capabilities
- **Phase 2 COMPLETE**: Full deployment pipeline live end-to-end
- **No coding sandbox** — replaced by future lean dev agent (TBD)

### Full Deployment Workflow (all working in sandbox)
- **flgh** — GitHub repo create, push, pull, PR management (no credentials needed in sandbox)
- **flvercel** — Vercel project/domain/deployment management
- **fldns** — Cloudflare DNS create, list, update (including proxy toggle)
- **First live deployment**: https://dashboard.test.forgelabs.nz
  - Modernize Admin Dashboard (Next.js 15 + MUI 7)
  - GitHub: forgelabs-sierra/sierra-dashboard (app moved from package/ subdir to repo root 2026-03-13)
  - Vercel project: `sierra-dashboard` (was `package`, recreated properly with GitHub link 2026-03-13)
  - DNS: dashboard.test.forgelabs.nz → cname.vercel-dns.com (DNS-only, no proxy)

### Key Lessons
- Cloudflare proxy must be OFF for Vercel custom domains (SSL won't work proxied)
- Sandbox $HOME is /workspace — exec tools run in sandbox always
- Gateway user is `node` (uid 1000), home /home/node (changed from openclaw/1001 on 2026-03-13)
- Git identity must be set per-repo in sandbox: sierra / sierra@forgelabs.nz
- After `flgh clone`, remote URL has token baked in — run `git remote set-url origin https://github.com/...` before flgh push
- Fetch articles with `curl https://r.jina.ai/<url>` — faster and more reliable than browser for article content

## Infrastructure (2026-03-13)
- **Gateway rebuilt** — Dockerfile.openclaw rewritten to align with upstream OpenClaw (pnpm, build:docker, NODE_ENV=production)
- **User:** openclaw (uid 1001) → node (uid 1000) — fixes workspace zeroing from uid mismatch
- **Paths:** /home/openclaw/ → /home/node/ everywhere
- **GitHub 2FA** enabled on forgelabs-sierra. Recovery codes in openclaw-podman/secrets/ (host only)
- **File zeroing bug fixed** — OpenClaw 2026.3.12 (f486859) — upstream atomic "pinned" write fix synced
- **Lost:** memory/2026-03-12.md was zeroed — rebuilt by Cameron from Telegram screenshots. MEMORY.md + TOOLS.md restored from git (b3d5e04)

## New Capabilities (2026-03-13)
- **`flvercel projects create`**: `flvercel projects create my-app --repo forgelabs-sierra/my-app [--framework auto]`

## Workspace Backup (2026-03-13)
- **Repo:** https://github.com/forgelabs-sierra/openclaw-workspace (private)
- **Skill:** /workspace/skills/workspace-backup/SKILL.md
- **Backs up:** MEMORY.md, TOOLS.md, HEARTBEAT.md, skills/, memory/
- **Secret handling:** GCP OAuth credentials auto-redacted from webdev skill before push
- **Heartbeat:** wired in — runs every heartbeat cycle
- **No-op when clean:** skips commit if nothing changed

## GitHub Orgs
- **forgelabs-nz** — company org (company repos, Matt + Cameron + Sierra collaborate)
- **forgelabs-sierra** — Sierra's personal account (workspace backup, openclaw-sierra config)
- **forgelabs-devops** — Cameron's account

## Forge Labs Website — Branch Workflow
- **Live site edits** (content, copy, quick fixes to www.forgelabs.nz) → commit directly to `main`, then merge into `feature/cms`
- **Development work** (new features, UI changes) → work in `feature/cms`, PR to `main` when ready

## Forge Labs Website v1.0.1 (2026-03-10)
- **Live at:** https://www.forgelabs.nz (production, main branch)
- **Preview:** https://sierrawins.test.forgelabs.nz (feature/cms branch)
- **GitHub:** forgelabs-sierra/forgelabs-web
- **Stack:** Next.js 15 + Tailwind v3 + shadcn + Auth.js v5 + Octokit
- **CMS:** /admin — split-pane editor, shortcode toolbar, image manager, SSR (updates on refresh)
- **GCP project:** fl-sparkyn (OAuth credentials)
- **GCP OAuth redirect:** ✅ `https://www.forgelabs.nz/api/auth/callback/google` added 2026-03-10
- **Matt's Telegram:** 8107496961 (@matmcf)

## Forge Labs Website (2026-03-10)
- **Live at:** https://sierrawins.test.forgelabs.nz
- **GitHub:** forgelabs-sierra/forgelabs-web
- **Stack:** Next.js 15 + Tailwind v3 + shadcn (v3-compat) + custom shortcode CMS
- **Vercel project:** `forgelabs-web` | Domain: sierrawins.test.forgelabs.nz → eventually forgelabs.nz
- **Contact form:** Google Apps Script (same endpoint as dv.forgelabs.nz) — uses FormData not JSON
- **GCP OAuth:** fl-sparkyn project, credentials in Vercel env vars
- **Admin CMS:** skeleton only — Auth.js + Octokit wired, but editor UI is placeholder
- **Design:** Dark hero (near-black + Unsplash space photo at 35% opacity), premium slate palette, font-black headings

## Technical Gotchas (Sandbox)
- **Tailwind v4 + arm64 = broken** — oxide native binary won't install, always use Tailwind v3
- **Screenshots:** cp from /data/browser-screenshots/ to /workspace/ before using image tool; wait 4-5s after navigate
- **Cloudflare proxy must be OFF** for Vercel custom domains (SSL breaks)
- **Google Apps Script:** expects FormData.append(), not JSON.stringify()
- **Vercel project creation:** `flvercel projects create my-app --repo forgelabs-sierra/my-app` (idempotent)
- **Article fetching:** use `curl https://r.jina.ai/<url>` — bypasses bot checks, no browser needed
- **flgh clone remote URL:** has token baked in after clone — fix with `git remote set-url origin https://github.com/org/repo.git`

## Timezone
- All timekeeping is in **Pacific/Auckland (NZDT)** — migrated 2026-03-11
- Earlier memory files (before ~11 March) may mix UTC and NZDT — 13h difference, can look like different days

## Key Protocols
1. **Tool Installation**: Request Cameron for system-level installs (don't improvise)
2. **Scope Expansion**: Ask Cameron to re-auth services when write access needed
3. **Evidence-based**: Always include confidence percentages in recommendations
4. **Project Tracking**: Monitor Cameron's 9+ parallel projects for dropped plates
5. **Context efficiency**: Use coding sandbox for development work to preserve main session
6. **Email protocol**: Always send from sierra@forgelabs.nz, always CC cameron@forgelabs.nz, always get explicit approval before sending. Skill: /workspace/skills/email/SKILL.md
7. **Telegram chunking**: Break long messages into short chunks — Telegram cuts off long messages