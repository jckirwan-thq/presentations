# Claude Code Setup — macOS Handout

> "I can only show you the door. You're the one that has to walk through it." — Morpheus

JC's Tradify-specific agentic workflow. Reproduce exactly. ~30 min one-time setup, compounds daily after.

**Audience:** Tradify Frontend engineers on macOS (Apple Silicon or Intel).
**Author:** Jean-Claude Kirwan · `jeanclaude.kirwan@theaccessgroup.com`
**Companion deck:** `index.html` (open in a browser).

---

## 0. Prerequisites

```bash
# Homebrew (skip if you have it):
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Node 20+ (Claude Code needs it):
brew install node
node --version   # v20+

# Git (you almost certainly have it):
git --version
```

---

## 1. Install Claude Code CLI

```bash
npm i -g @anthropic-ai/claude-code
claude --version
claude            # first run launches browser auth flow
```

---

## 2. Folder layout — the parent `tradify/` trick

```bash
mkdir -p ~/development/tradify
cd ~/development/tradify

# Clone the 4 repos as siblings:
git clone git@github.com:TradifyHQ/tradify-frontend-platform.git
git clone git@github.com:TradifyHQ/web-frontend-angularjs.git
git clone git@github.com:TradifyHQ/web-platform.git
git clone git@github.com:TradifyHQ/tradify-shared-api.git
```

### CLAUDE.md — two options

The parent folder needs a `CLAUDE.md` (cross-repo brain) and an `AGENTS.md` (mirror for non-Claude agents). Pick one:

- **Option A (preferred): copy JC's curated files** from the playbook §3 or from JC's Obsidian vault. They already encode the conventions, MCP wiring, agent roster, and Jira workflow. Drop into `~/development/tradify/CLAUDE.md` and `~/development/tradify/AGENTS.md`.

- **Option B: bootstrap with `/init`.** Inside a Claude session opened from the folder, type the `/init` slash command. Claude analyses the codebase and writes a baseline `CLAUDE.md`. Useful starting point — then tune it. Note: this overwrites any existing file.

> `tradify-agent-skills/setup.sh` does NOT create `CLAUDE.md`. That script handles agent + skill symlinks only (see §7).

Open Claude from the parent and it picks up cross-repo context. Open it from inside one repo and it gets stack-specific context only.

```bash
cd ~/development/tradify && claude
# Inside the session, optional:
#   /init      → auto-generate CLAUDE.md (Option B)
```

---

## 3. ICM — persistent cross-session memory

```bash
brew install rtk-ai/tap/icm
icm --version
```

### Mandatory usage protocol

**Recall before starting work:**

```bash
icm recall "TRA-23186"
icm recall "TRA-23186" -t "context-tradify"
icm recall-context "TRA-23186" --limit 5
```

**Store when ANY of these happen** (do it before responding to the user, not later):

| Trigger | Command |
|---|---|
| Error resolved | `icm store -t errors-resolved -c "<root cause + fix>" -i high -k "kw1,kw2"` |
| Architecture decision | `icm store -t decisions-<project> -c "<decision + reason>" -i high` |
| User preference discovered | `icm store -t preferences -c "<dev preference>" -i critical` |
| Significant task completed | `icm store -t context-<project> -c "<work summary>" -i high` |
| 20+ tool calls without a store | drop a progress summary |

Topics JC keeps separate: `context-tradify`, `context-tradify-frontend-platform`, `preferences`, `errors-resolved`, `decisions-react-migration`, `references-external`.

Audit weekly: `icm health`.

---

## 4. `rtk` — Rust Token Killer

Rewrites routine bash via a hook → 60-90% token savings.

```bash
brew install rtk-ai/tap/rtk
rtk --version
rtk gain         # analytics, must NOT error
```

**Name collision warning:** avoid `reachingforthejack/rtk` (Rust Type Kit). If `rtk gain` errors with "command not found", check `which rtk`.

---

## 5. `agent-deck` — terminal session manager

```bash
brew install asheshgoplani/tap/agent-deck
agent-deck --version
agent-deck            # opens TUI
```

Dependency: `tmux` (brew installs it automatically).

Use for: multiple parallel Claude sessions, MCP attach/detach, fork sessions, worktree management, session export/import to share with teammates.

---

## 6. Plugins (install inside Claude with `/plugin install`)

High-value subset to install first:

```
/plugin install atlassian
/plugin install figma
/plugin install chrome-devtools-mcp
/plugin install slack
/plugin install context7
/plugin install superpowers
/plugin install caveman
/plugin install pr-review-toolkit
```

After those settle, add as you hit the use case:

```
agent-deck · claude-code-setup · claude-md-management · code-simplifier
frontend-design · github · n1-optimizer · playwright · security-guidance
skill-creator · typescript-lsp
```

OAuth/MCP auth flows open in your default browser when first invoked.

---

## 7. Specialist agents — symlinked from `tradify-agent-skills`

All agents and skills live in the `tradify-agent-skills` repo. `setup.sh` creates three symlinks:

| Symlink | Target |
|---|---|
| `~/.claude/agents` | `tradify-agent-skills/agents/` (all 8 tech-lead `.md` files) |
| `~/.claude/skills` | `tradify-agent-skills/` (skills live at repo top level) |
| `tradify/.claude/agents/*.md` | repo-level agents from `tradify-frontend-platform/.claude/agents/*.md` |

```bash
cd ~/development/tradify
git clone git@github.com:TradifyHQ/tradify-agent-skills.git
cd tradify-agent-skills
./setup.sh                 # global + project symlinks
# or scope it:
./setup.sh --global        # only ~/.claude
./setup.sh --project       # only parent .claude
```

Verify:

```bash
ls -la ~/.claude/agents ~/.claude/skills
# both should resolve to the tradify-agent-skills repo
```

> **What setup.sh does NOT do:** it does not create or touch `CLAUDE.md`. See §2 for CLAUDE.md options (copy curated file, or run `/init` in-session).

### Global tech leads (in `tradify-agent-skills/agents/`)

- `engineering-manager` — cross-repo architecture, migration planning
- `frontend-tech-lead` — React 19, Module Federation, Tailwind
- `backend-tech-lead` — .NET 10 + 4.7.2, EF, multi-tenancy
- `angularjs-tech-lead` — Legacy AngularJS, postMessage interop
- `ux-tech-lead` — Design system, Figma, accessibility, migration UX parity
- `e2e-testing-tech-lead` — Cross-repo test strategy, coverage gaps
- `devops-engineer` — Docker, Azure, CI/CD, local dev stack
- `security-reviewer` — Multi-tenant data isolation, auth, CSRF, OWASP

### Repo-specific agents

Live in `tradify-frontend-platform/.claude/agents/` — pulled with `git clone`. Includes `bundle-performance-engineer`, `code-reviewer`, `devx-engineer`, `docs-engineer`, `frontend-architect-ui`, `frontend-context-architect`, `platform-engineer`.

Updating any agent: edit in `tradify-agent-skills`, commit, push. Symlinks pick up the change everywhere immediately.

**Workflow rule:** never implement directly. Delegate to the specialist agent for the domain.

---

## 8. Hooks (`~/.claude/settings.json`)

After installing caveman + rtk + icm + agent-deck, your settings.json `hooks` section ends up like this (paths shown for macOS; adjust to your install locations):

```jsonc
{
  "hooks": {
    "SessionStart": [
      { "type": "command", "command": "node ~/.claude/hooks/caveman-activate.js" },
      { "type": "command", "command": "icm hook start" },
      { "type": "command", "command": "agent-deck hook-handler SessionStart" }
    ],
    "SessionEnd":   [{ "type": "command", "command": "icm hook end" },
                     { "type": "command", "command": "agent-deck hook-handler SessionEnd" }],
    "UserPromptSubmit": [
      { "type": "command", "command": "node ~/.claude/hooks/caveman-mode-tracker.js" },
      { "type": "command", "command": "icm hook prompt" },
      { "type": "command", "command": "agent-deck hook-handler UserPromptSubmit" }
    ],
    "PreToolUse": [
      { "matcher": "Bash", "hooks": [{ "type": "command", "command": "rtk hook claude" }] }
    ],
    "PreCompact": [{ "type": "command", "command": "icm hook compact" }],
    "Stop":       [{ "type": "command", "command": "agent-deck hook-handler Stop" }],
    "Notification":      [{ "type": "command", "command": "agent-deck hook-handler Notification" }],
    "PermissionRequest": [{ "type": "command", "command": "agent-deck hook-handler PermissionRequest" }]
  }
}
```

### Caveman mode — the single biggest leverage

```
/caveman lite     # mild
/caveman full     # JC's default
/caveman ultra    # maximum compression
```

To disable mid-session: type "stop caveman" or "normal mode". Code blocks, commits, PRs, security warnings stay normal — only prose gets compressed.

---

## 9. Obsidian Vault — PARA layout + `plans/` symlink

### PARA structure

PARA (Tiago Forte): **P**rojects · **A**reas · **R**esources · **A**rchives. JC uses a Johnny-Decimal-flavoured PARA:

```bash
mkdir -p "$HOME/Documents/Obsidian Vault"
cd "$HOME/Documents/Obsidian Vault"

mkdir -p \
  20-meetings \
  30-projects/react-migration/plans \
  40-hiring/scorecards \
  50-references \
  60-archive \
  99-MISC \
  _templates
```

| Folder | Maps to | What lives here |
|---|---|---|
| `20-meetings/` | Projects/Resources hybrid | Standups, steering committee, 1:1s |
| `30-projects/` | **Projects** | Active work: react-migration, TRA-XXXXX plans |
| `40-hiring/` | **Areas** | Ongoing area of responsibility (scorecards, candidates) |
| `50-references/` | **Resources** | Permanent reference docs (this playbook lives here) |
| `60-archive/` | **Archives** | Completed projects, retired notes |
| `99-MISC/` | inbox | Capture-then-sort |
| `_templates/` | — | Templated notes (sprint review, ADR, PR template) |

Open the vault in Obsidian → Settings → Files & links → enable "Default location for new notes: same folder as current file".

### `plans/` symlink — commit plans with code

```bash
cd ~/development/tradify/tradify-frontend-platform
ln -s "$HOME/Documents/Obsidian Vault/30-projects/react-migration/plans" plans
ls -la plans       # confirm symlink target
```

Plans now live in Obsidian (full markdown editor, backlinks, graph view) but commit with the repo. Best of both worlds.

### Don't use Obsidian?

Plans are just `.md` files. Point the symlink at any folder you maintain in any editor: VS Code, iA Writer, Typora, Logseq, plain text. The repo doesn't care.

---

## 10. Conventions stored as memory

Memory dir on macOS:

```bash
~/.claude/projects/-Users-<your-mac-username>-development-tradify/memory/
```

JC's "always / never" rules to seed yours:

- Never push to main directly — always branch + PR
- Never implement directly — delegate to specialist agent
- PR title: `TRA-NUMBER - feature/bucket/name` (e.g. `TRA-23186 - feature/devx/opt-in pre-push Claude agent review hook`)
- No secrets in commits, logs, or output
- `tradify-frontend-platform` uses Jest, NOT Vitest
- Named Tailwind utilities (`font-light`) not arbitrary (`font-weight-[300]`)
- Use named + default exports (not named-only)
- TS 3.4.5 in `web-frontend-angularjs`: no `?.`, no `??` — explicit null checks
- Extract components to own files — never inline multiple in one file
- Verify bundle size after every Rspack / MF / dependency change
- Sync your primary local checkout to the new SHA BEFORE pushing from a worktree
- MFE plugin > shared package for anything with independent deploy needs
- On QA Ready transition: attach current open release `fixVersion`
- Jira auto-transitions to Code Review on PR open — don't revert it
- No invented stakeholders — use role descriptors when uncertain
- No acronyms in user-facing text

Easiest way to add one: in a session say "remember that X" and Claude writes the memory file.

---

## 11. Pre-push agent-review hook (TRA-23186)

Already lives in `tradify-frontend-platform/.husky/pre-push` after you clone.

```bash
cd ~/development/tradify/tradify-frontend-platform
# the platform repo uses bun — package install triggers husky setup:
bun install
```

Use it:

```bash
git push                         # tests + coverage only
git push -o agent-review         # also: parallel Claude agent review
git agent-push                   # alias for the above (set up via git config)
SKIP_PRE_PUSH_TESTS=1 git push   # emergency hotfix only
```

If `claude` CLI is absent, the hook warns and lets the push through — never blocks non-Claude teammates.

---

## 12. Verify your setup

```bash
claude --version          # CLI installed
icm recall "test"         # ICM connected
rtk gain                  # rtk wired
agent-deck --version      # session manager
ls -la ~/.claude/agents/  # symlink → tradify-agent-skills/agents
cd ~/development/tradify && claude   # cross-repo brain loads
```

Inside `tradify-frontend-platform/` only: `bun --version` should also work (it's the package manager that repo uses).

Inside Claude session, type `/plugin` → see installed plugins. Type `/caveman full` → confirm activation.

---

## 13. Day-1 onboarding checklist

- [ ] Install Claude Code + icm + rtk + agent-deck
- [ ] Clone all 4 Tradify repos + `tradify-agent-skills` as siblings under `~/development/tradify/`
- [ ] Drop in parent `CLAUDE.md` + `AGENTS.md`
- [ ] Run `tradify-agent-skills/setup.sh` (symlinks all agents/skills globally + project)
- [ ] Install 8 high-value plugins
- [ ] Authenticate Atlassian + Figma + Slack + Chrome DevTools MCPs
- [ ] Symlink `plans/` to your Obsidian vault (if applicable)
- [ ] Run `/caveman full` and notice the output difference
- [ ] On your first PR, use `TRA-XXXXX - feature/bucket/name` title
- [ ] Add one `icm store` entry by EOW1
- [ ] Ping JC on Slack with anything that should be formalized as a feedback memory

---

## 14. Things to skip on a fresh install

- Don't enable every plugin from day 1 — add as you hit the use case
- Don't write `CLAUDE.md` by hand — use `claude-md-management:claude-md-improver`
- Don't use `--force-push`; always `--force-with-lease`
- Don't `git stash` when a rebase is what you want
- Don't set telemetry env vars unless your org provides an OTLP endpoint

---

## 15. Trouble?

- `claude` not auth'd → re-run `claude` to refresh browser flow
- ICM "command not found" → `brew reinstall rtk-ai/tap/icm`; check `which icm`
- rtk filtering breaks a command → `rtk proxy <cmd>` runs raw
- Husky hook not firing → `rm -rf .husky/_ && bun install` to re-init
- MCP auth loops → revoke + re-authenticate from the third-party (Atlassian/Slack/Figma)

> "Remember, all I'm offering is the truth." — Morpheus

Slack: `#frontend` · JC handles convention reconciliation at the next steering committee.
