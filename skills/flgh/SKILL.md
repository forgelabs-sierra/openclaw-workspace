---
name: flgh
description: Create and manage GitHub repos, PRs, issues, releases, and handle git push/pull/fetch/clone for forgelabs projects. All operations go through the host API — no credentials needed in sandbox.
metadata: {"clawdbot":{"emoji":"","requires":{"bins":["flgh"]}}}
---

# flgh — GitHub & Git Transport for Sandbox

Manages GitHub resources and authenticated git operations via the host clitools API.
No GitHub credentials exist in the sandbox — all auth is handled on the host.

## When to use

- Creating GitHub repos for new projects
- Creating and managing pull requests
- Pushing/pulling code to/from GitHub
- Cloning repos into the workspace
- Setting up git remotes
- Creating releases and managing issues

## Important: Security

- **Create, Read, and Update** for GitHub resources (repos, PRs, issues, releases)
- Repo metadata can be updated (description, homepage, topics, visibility)
- You **cannot** delete repos, close PRs, close issues, or delete releases
- If you need a destructive operation, ask the operator to do it
- **Always ask the user** before creating repos or PRs — confirm the name, visibility, and details

## Repository commands

```bash
flgh repo list                              # list your repos
flgh repo create my-project                 # create public repo
flgh repo create my-project --private       # create private repo
flgh repo create my-project --desc "About"  # with description

flgh repo update --desc "New description"   # update repo (auto-detects from .git)
flgh repo update owner/repo --desc "Desc"   # explicit repo
flgh repo update --homepage "https://example.com"
flgh repo update --topics "ai,nextjs,web"   # comma-separated topics
flgh repo update --private                  # make private
flgh repo update --public                   # make public
flgh repo update --desc "Desc" --homepage "https://example.com" --topics "ai,web"
```

## Pull request commands

```bash
flgh pr list                                # list open PRs (auto-detects repo)
flgh pr list --state all                    # include closed
flgh pr list --repo forgelabs/my-project    # explicit repo

flgh pr create --title "Add feature" --head feature/xyz --base main
flgh pr create --title "Fix" --head fix/bug --body "Details here"

flgh pr view 42                             # view PR details
flgh pr merge 42                            # merge PR
flgh pr merge 42 --method squash            # squash merge
```

## Issue commands

```bash
flgh issue list                             # list open issues
flgh issue list --state all --repo owner/name

flgh issue create --title "Bug report" --body "Description"
```

## Release commands

```bash
flgh release list
flgh release create v1.0.0 --title "First release" --body "Notes"
flgh release create v1.1.0-beta --prerelease
```

## Git transport commands

These run git operations on the host where credentials exist. Use these instead of `git push`/`git pull` directly.

```bash
flgh push                                   # push current branch to origin
flgh push --branch main                     # push specific branch
flgh push --force                           # force push
flgh push --tags                            # push tags
flgh push -u --branch feature/xyz           # push and set upstream tracking

flgh pull                                   # pull from origin
flgh pull --branch main                     # pull specific branch

flgh fetch                                  # fetch from origin
flgh fetch --remote upstream                # fetch from different remote

flgh clone forgelabs/my-project             # clone into /workspace/my-project
flgh clone forgelabs/my-project my-dir      # clone into /workspace/my-dir
```

## Remote management

```bash
flgh remote add forgelabs/my-project        # add origin -> github.com/forgelabs/my-project
flgh remote add forgelabs/my-project --remote upstream  # add as 'upstream'

flgh remote set-url forgelabs/other-project # update origin URL

flgh remote remove                          # remove origin
flgh remote remove --remote upstream        # remove upstream
```

## Typical workflows

### New project: create repo and push

```bash
git init && git add . && git commit -m "initial commit"
flgh repo create my-project --desc "My new project"
flgh remote add forgelabs/my-project
flgh push -u --branch main
```

### Feature branch: create PR

```bash
git checkout -b feature/xyz
# ... make changes ...
git add . && git commit -m "add feature xyz"
flgh push -u --branch feature/xyz
flgh pr create --title "Add feature xyz" --head feature/xyz --base main
```

### Sync with remote

```bash
flgh fetch
git rebase origin/main
flgh push
```

### Clone existing repo

```bash
flgh clone forgelabs/existing-project
cd /workspace/existing-project
```

## What works locally (no flgh needed)

These git commands work directly in the sandbox without flgh:
`git init`, `git add`, `git commit`, `git branch`, `git checkout`, `git merge`,
`git rebase`, `git stash`, `git log`, `git diff`, `git status`, `git tag`

## Auto-detection

For PR, issue, and release commands, flgh auto-detects the repo from `.git/config`
in the current directory. Use `--repo owner/name` to override or when not in a git directory.

## Limitations

- **No destructive operations** — cannot delete repos, close PRs/issues, or delete releases
- **Repo update is metadata only** — can change description, homepage, topics, visibility; cannot rename or transfer
- **GitHub only** — remote URLs must be github.com (security: prevents token exfiltration)
- **Workspace paths only** — git transport commands only work on /workspace/ paths
