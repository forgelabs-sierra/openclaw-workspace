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
- **Next.js versions:** Do NOT hardcode specific versions — use `"next": "latest"` or check npm tags before pinning. **15.2.2 had React2Shell CVE** — fixed in 15.2.3+. Safe versions: >=15.2.3, latest stable 15.x is 15.2.9 / 15.3.9. Current latest: 16.1.6. Always use the latest patch release.
- **`create-next-app` is interactive** — pass all flags to skip prompts: `npx create-next-app@latest my-app --yes --typescript --tailwind --eslint --app --src-dir --no-import-alias --no-turbopack`. Without these flags, it blocks on interactive prompts that can't be answered from exec.
- **Screenshots:** use `/data/browser-screenshots/` path (bind-mounted into sandbox) — NOT `/home/node/.openclaw/media/browser/` (outside workspace, image tool will error)
- **Cloudflare proxy must be OFF** for Vercel custom domains (SSL breaks)
- **Google Apps Script:** expects FormData.append(), not JSON.stringify()
- **Vercel project creation:** `flvercel projects create my-app --repo forgelabs-sierra/my-app` (idempotent)
- **Article fetching:** use `curl https://r.jina.ai/<url>` — bypasses bot checks, no browser needed
- **flgh clone remote URL:** has token baked in after clone — fix with `git remote set-url origin https://github.com/org/repo.git`
- **Temp files:** file tools (read/write/edit) are workspace-only — use `exec` for `/tmp/` access. For temp files accessible via file tools, use `/workspace/.tmp/`
- **sessions_spawn:** do NOT use `streamTo: "parent"` — only works with acp runtime, not subagent. Check spawned sessions via sessions_list/sessions_history instead.

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

## Local AI & LiteLLM Integration (2026-03-15)

### The DGX Spark
The DGX Spark is an NVIDIA GB10 ARM64 machine with 120GB unified VRAM, sitting on the local network at Tailscale IP 100.81.205.107 (hostname: `aitopatom-3d7c`). It's Cameron's self-hosted GPU server for running large language models locally — no API costs, no rate limits, full control.

### Timeline

**March 7** — LiteLLM proxy setup
Cameron set up the LiteLLM proxy as a credential broker between me (OpenClaw) and Anthropic's API. This was the foundation — I talk to LiteLLM at localhost:4000, and LiteLLM handles upstream auth. Five bugs were found and fixed during initial setup (OOM kills, enterprise-only config, env var routing, model name mismatches, under-resourced Podman machine).

**March 7** — Z.AI provider switching (Claude Code)
First provider switching was set up for Claude Code (the host tool Cameron uses), not for me. Claude Code's `~/.claude/settings.json` was configured to swap between Anthropic direct and Z.AI (GLM models). Z.AI provides Anthropic-compatible models: `glm-5` (opus-tier), `glm-4.7` (sonnet-tier), `glm-4.5-air` (haiku-tier). Backup configs saved.

**March 9** — Token optimisation research
Cameron researched using a local model for my heartbeat and cron jobs to reduce Anthropic API token costs. The heartbeat runs hourly (reads HEARTBEAT.md, checks email, runs checklists) and the email cron runs every 10 minutes — these don't need frontier intelligence.

**March 11** — DGX Spark goes online with GLM-4.7 Flash
The DGX started serving `cyankiwi/GLM-4.7-Flash-AWQ-4bit` via vLLM on port 8000. It was added to LiteLLM as `glm-local-heartbeat` — my heartbeat and cron jobs switched to use this free local model instead of paid Anthropic Haiku. Tool calling was verified working (direct, and through LiteLLM's translation layer).

**Key discovery:** vLLM serves an OpenAI-compatible API (not Anthropic), so LiteLLM uses `openai/` prefix internally — but LiteLLM translates between formats transparently.

**March 11** — GLM session corruption incident
The GLM-4.7 Flash model generated malformed tool call responses during a heartbeat turn. These corrupt entries got written into my main session JSONL file. The Anthropic API then rejected the entire conversation context because of the malformed entries, making me unresponsive on Telegram. Resolution: truncated the corrupt lines from the session file. **Lesson:** heartbeat and cron were moved to their own isolated sessions so a local model failure can never corrupt the main conversation.

**March 14** — Z.AI integration for OpenClaw via LiteLLM
The big change. Cameron built transparent provider switching for ME (not just Claude Code). Three separate LiteLLM config files were created — `litellm-config-anthropic.yaml`, `litellm-config-zai.yaml`, and the active `litellm-config.yaml`. A switch script (`scripts/switch-provider.sh`) handles the full shutdown/swap/restart sequence in 6 steps.

**The critical bug:** LiteLLM's BYOK (Bring Your Own Key) feature was passing my OAuth token through to Z.AI (which rejected it). The fix was changing my `apiKey` in `openclaw.json` from `${ANTHROPIC_OAUTH_TOKEN}` to `${ANTHROPIC_API_KEY}` (which equals the LiteLLM master key). LiteLLM then recognises me as a proxy admin and uses the correct per-model API key from its config for upstream auth.

All upstream authentication was moved into LiteLLM config — I only know the master key, LiteLLM handles the rest.

**March 14** — First Z.AI model behaviour difference
My first memory write of the day failed because the Z.AI GLM model used edit on a file that didn't exist yet (2026-03-14.md). I self-recovered by falling back to write. This is a minor behaviour difference — Claude uses write for new files, GLM tries edit first. One wasted tool call per day, harmless.

**March 14–15** — Qwen3-Coder-Next-FP8 on DGX Spark via sglang
Cameron set up a much more capable model on the DGX: `Qwen3-Coder-Next-FP8` (80B params, 3B active via MoE, FP8 quantised). This is a hybrid Gated DeltaNet + Attention + MoE architecture — not a standard transformer. It's served by sglang (not vLLM) on port 30000.

**Key specs:**
- 131K context window (50% of native 256K, balanced against GPU memory)
- 58 concurrent requests capacity
- 45 tokens/sec single-request, 141 tokens/sec at concurrency 8
- Tool calling via `--tool-call-parser qwen3_coder`
- Triton attention backend (mandatory on Blackwell GPUs)
- ~5 minute startup (dominated by loading 75GB of model weights)

Speculative decoding (EAGLE3 with Aurora-Spec draft model) was tested but rejected: only 18% speedup, limits concurrency to 1 request, and forces 32K context. Baseline wins for agent workloads.

**March 15** — DGX integrated as third LiteLLM provider
Cameron created `litellm-config-dgx.yaml` and updated the switch script. Verified the LiteLLM container can reach the DGX over Tailscale. Switched to DGX provider, health check passed (7/7 healthy), end-to-end inference confirmed (`claude-sonnet-4-6` → LiteLLM → sglang → `Qwen3-Coder-Next` → response).

### Current State: Three Providers

```bash
./scripts/switch-provider.sh anthropic  # Anthropic Claude API (production)
./scripts/switch-provider.sh zai        # Z.AI GLM models (Anthropic-compatible)
./scripts/switch-provider.sh dgx        # DGX Spark Qwen3-Coder-Next (self-hosted)
```

I see the same model names regardless of which backend is active:

| What I request | Anthropic | Z.AI | DGX Spark |
|----------------|-----------|------|-----------|
| `claude-sonnet-4-6` | Claude Sonnet 4.6 | GLM-4.7 | Qwen3-Coder-Next-FP8 |
| `claude-opus-4-6` | Claude Opus 4.6 | GLM-5 | Qwen3-Coder-Next-FP8 |
| `claude-haiku-4-5` | Claude Haiku 4.5 | GLM-4.5-Air | Qwen3-Coder-Next-FP8 |

**Note:** DGX Spark has only one model — all names route to the same `Qwen3-Coder-Next-FP8`.

### Architecture

Me (OpenClaw) → LiteLLM proxy (localhost:4000) → Provider
├── Anthropic API (cloud)
├── Z.AI API (cloud)
└── DGX Spark sglang (Tailscale, self-hosted)

LiteLLM handles:
- API format translation (OpenAI ↔ Anthropic)
- Credential management (I only know the master key)
- Model name routing (same names, different backends)
- Health checks, rate limiting, cost tracking

### Key Lessons Learned

1. **LiteLLM BYOK** — If you send a key that isn't the master key, LiteLLM passes it through to the upstream provider (overriding config). Always authenticate with the master key.

2. **Local models can corrupt sessions** — The GLM-4.7 Flash incident showed that local models with different tool call formats can inject malformed data into session files, breaking subsequent API calls. Always use isolated sessions for non-frontier models.

3. **sglang vs vLLM** — sglang is the recommended framework for Qwen3-Coder-Next. vLLM can't do EAGLE3 speculative decoding with this architecture. sglang handles the hybrid GDN model correctly.

4. **Speculative decoding trade-offs** — On the GB10, EAGLE3 gives only 18% speedup but kills concurrency (58 → 1) and halves context (131K → 32K). Baseline is better for agent workloads.

5. **Podman containers CAN reach Tailscale IPs** — The `openclaw-external` bridge network routes to the host's Tailscale interface. No special configuration needed.

6. **podman restart doesn't pick up env changes** — Must `podman rm` + `podman compose up -d` for docker-compose.yml environment changes.

7. **Model behaviour differences matter** — Z.AI's GLM uses edit before write for new files. Qwen3-Coder-Next may have its own quirks. Monitor after switching.

## Session Best Practices
- **exec tool host:** Always use `host: "sandbox"` or omit host (defaults to sandbox). Never set `host: "gateway"` — commands run in sandbox container.
- **Memory files:** Use `write` for new daily files, `edit` only for existing files. Don't read first — just write.
- **Read offsets:** Check file length with `wc -l <file>` via exec before using offset to avoid `ENOENT` errors.
- **CLI tools:** `flgog`, `flzotero`, `flvercel`, `flgh`, `fldns` are CLI commands — always run via `exec`, not as standalone tools.

### When to Switch Providers

**Switch to Anthropic when:**
- Task is difficult, stuck in a loop, or model isn't following instructions
- Need maximum intelligence/reasoning (Claude Opus 4.6: 80.8% SWE-bench, 65.4% Terminal-Bench)
- Best instruction following, most reliable tool calling, deepest reasoning
- "I've tried multiple approaches and none are working" — this is the move
- Token budget not constrained (Cameron may switch away if approaching daily/weekly limits)

**Switch to Z.AI when:**
- Routine work where cost matters (GLM-5: 77.8% SWE-bench)
- Good multilingual coding and web browsing
- "Preserved thinking" needed — keeps reasoning consistent across long sessions
- Large tasks where Cameron wants cheaper pricing
- "Try multiple approaches cheaply" — run parallel explorations
- Token budget constrained — cheaper than Anthropic

**Switch to DGX when:**
- Want to run parallel sub-agents (58 concurrent requests, zero cost)
- Tasks can be split: write code, tests, docs simultaneously
- "Try multiple solution approaches in parallel"
- Iterating freely without worrying about API costs
- Simple tasks where local execution is sufficient
- Long-running tasks that would exhaust token allowance on cloud providers
- Privacy matters — all data stays local (no external API calls)
- Expected benefit: 2.3x more output at concurrency 4, scales further

**Cameron's practical switching factors:**
- Cost: Z.AI cheaper than Anthropic, DGX is free
- Token limits: May switch off Anthropic when approaching daily/weekly limits, even if it's best tool
- Task type: DGX for simple/long-running, Anthropic for complex
- Privacy: DGX for sensitive tasks (all data stays local)
- Parallel work: All three support parallel sub-agents; Anthropic and Z.AI may be better (untested)

### Provider Capabilities Summary

| What I request | Anthropic | Z.AI | DGX Spark |
|----------------|-----------|------|-----------|
| claude-sonnet-4-6 | Claude Sonnet 4.6 | GLM-4.7 | Qwen3-Coder-Next |
| claude-opus-4-6 | Claude Opus 4.6 | GLM-5 | Qwen3-Coder-Next |
| claude-haiku-4-5 | Claude Haiku 4.5 | GLM-4.5-Air | Qwen3-Coder-Next |

**Note:** DGX Spark has only one model — all names route to the same Qwen3-Coder-Next. Z.AI maps different GLM models to each tier.

### Relevant Quicknotes
| Date | File | Topic |
|------|------|-------|
| Mar 7 | quicknote_20260307_193017_litellm-setup.md | Initial LiteLLM proxy setup (5 bugs found) |
| Mar 7 | quicknote_20260307_164200_llm-provider-switching.md | Z.AI/Anthropic switching for Claude Code |
| Mar 9 | quicknote_20260309_local-llm-heartbeat-token-optimisation.md | Local model for heartbeat research |
| Mar 11 | quicknote_20260311_150000_dgx-spark-glm-local-model.md | DGX Spark GLM-4.7 Flash setup, tool calling verified |
| Mar 11 | quicknote_20260311_164500_glm-session-corruption.md | GLM corrupted main session incident |
| Mar 14 | quicknote_20260314_083822_zai-litellm-model-switching.md | Z.AI switching plan for OpenClaw |
| Mar 14 | quicknote_20260314_095848_zai-litellm-implementation.md | Z.AI implementation, BYOK bug, validation |
| Mar 14 | quicknote_20260314_105140_zai-glm-edit-vs-write-behaviour.md | Z.AI model edit vs write behaviour |
| Mar 14 | quicknote_20260314_221921_openclaw_litellm_integration.md | Qwen3-Coder-Next + OpenClaw + LiteLLM research |
| Mar 14 | quicknote_20260314_222414_dgx-spark-litellm-provider-integration.md | DGX as third LiteLLM provider plan |
| Mar 15 | quicknote_20260315_154024_definitive_setup_guide.md | Definitive sglang setup guide (benchmarks, memory, flags) |
