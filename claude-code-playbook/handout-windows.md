# Claude Code Setup — Windows 11 Handout

> "Welcome to the desert of the real." — Morpheus

JC's Tradify-specific agentic workflow on native Windows 11.

**Audience:** Tradify Frontend engineers on Windows 11.
**Author:** Jean-Claude Kirwan · `jeanclaude.kirwan@theaccessgroup.com`
**Companion deck:** `index.html` (open in a browser).

> WSL2 is an option if you want POSIX-shell parity for husky/setup.sh, but everything below works in native PowerShell.

---

## 0. Prerequisites

```powershell
# Node 20+:
winget install OpenJS.NodeJS.LTS
node --version

# Git (if not already):
winget install Git.Git
git --version

# Windows Terminal (much better than cmd.exe):
winget install Microsoft.WindowsTerminal

# Optional but very useful:
winget install Microsoft.PowerShell    # PowerShell 7+

# Scoop — light package manager used below:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex
```

Enable **Developer Mode** (Settings → Privacy & security → For developers) so symlinks work without admin elevation.

---

## 1. Install Claude Code CLI

```powershell
npm i -g @anthropic-ai/claude-code
claude --version
claude            # first run launches browser auth flow
```

If `npm i -g` warns about permissions, install Node via the official `.msi` rather than Scoop/Chocolatey.

---

## 2. Folder layout — the parent `tradify\` trick

```powershell
mkdir $HOME\development\tradify
cd $HOME\development\tradify

git clone git@github.com:TradifyHQ/tradify-frontend-platform.git
git clone git@github.com:TradifyHQ/web-frontend-angularjs.git
git clone git@github.com:TradifyHQ/web-platform.git
git clone git@github.com:TradifyHQ/tradify-shared-api.git
```

### CLAUDE.md — two options

The parent folder needs a `CLAUDE.md` (cross-repo brain) and an `AGENTS.md` (mirror for non-Claude agents). Pick one:

- **Option A (preferred): copy JC's curated files** from the playbook §3. Drop them into `$HOME\development\tradify\CLAUDE.md` and `$HOME\development\tradify\AGENTS.md`. They already encode conventions, MCP wiring, agent roster, and Jira workflow.

- **Option B: bootstrap with `/init`.** Inside a Claude session opened from the folder, type the `/init` slash command. Claude analyses the codebase and writes a baseline `CLAUDE.md`. Useful starting point — then tune it. Overwrites any existing file.

> `tradify-agent-skills\setup.ps1` does NOT create `CLAUDE.md`. That script handles agent + skill symlinks only (see §7).

Open Claude from the parent for cross-repo context:

```powershell
cd $HOME\development\tradify
claude
# Inside the session, optional:
#   /init      → auto-generate CLAUDE.md (Option B)
```

**Note on case sensitivity:** Claude treats `.claude` and `CLAUDE.md` as the canonical names. Don't let Explorer auto-capitalize them. Keep `.claude` lowercase.

---

## 3. ICM — persistent cross-session memory

No Homebrew on Windows, so use Cargo or grab a release binary.

### Option 1 — Cargo (needs Rust toolchain)

```powershell
# Install rustup once:
winget install Rustlang.Rustup
# New shell, then:
cargo install icm-cli
icm --version
```

### Option 2 — Release binary

Download from `https://github.com/rtk-ai/icm/releases`, extract to `$HOME\.local\bin\`, add that to PATH.

### Mandatory usage protocol

Same as macOS — recall before work, store on five triggers (errors resolved, decisions, preferences, significant tasks, every 20 tool calls):

```powershell
icm recall "TRA-23186"
icm recall "TRA-23186" -t "context-tradify"
icm recall-context "TRA-23186" --limit 5

icm store -t errors-resolved -c "<root cause + fix>" -i high -k "kw1,kw2"
icm store -t decisions-react-migration -c "<decision + reason>" -i high
icm store -t preferences -c "<dev preference>" -i critical
icm store -t context-tradify -c "<work summary>" -i high

icm health
```

---

## 4. `rtk` — Rust Token Killer

```powershell
cargo install rtk-cli
# OR binary release: https://github.com/rtk-ai/rtk/releases

rtk --version
rtk gain          # must NOT error
```

**Name collision warning:** `reachingforthejack/rtk` (Rust Type Kit) is a different tool. If `rtk gain` fails with "command not found", check `where.exe rtk`.

---

## 5. `agent-deck` — terminal session manager

No Homebrew on Windows. Grab the prebuilt binary from the GitHub releases page:

```powershell
# Releases: https://github.com/asheshgoplani/agent-deck/releases
# Download the Windows .zip, extract, and put agent-deck.exe on PATH.
# Easiest path:
mkdir $HOME\.local\bin -ErrorAction SilentlyContinue
# (extract agent-deck.exe to $HOME\.local\bin)
$env:Path += ";$HOME\.local\bin"     # for this session
# Persist:
[Environment]::SetEnvironmentVariable("Path", "$env:Path", "User")

agent-deck --version
agent-deck         # opens TUI — use Windows Terminal for best rendering
```

Dependency: `tmux`. Windows native tmux is via MSYS2 or Git Bash; alternatively run agent-deck from inside WSL2 if you want it without ceremony.

---

## 6. Plugins (install inside Claude with `/plugin install`)

Same high-value subset as macOS:

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

**Windows-specific notes:**

- **Chrome DevTools MCP**: needs Chrome installed. If `chrome.exe` is not in PATH, set `CHROME_PATH` env var to `C:\Program Files\Google\Chrome\Application\chrome.exe`.
- **Playwright** (if you add it): run `npx playwright install` once, then `npx playwright install-deps` is unnecessary on Windows.
- **OAuth flows**: open in your default browser. If you use Edge as default, that's fine.

---

## 7. Specialist agents — symlinked from `tradify-agent-skills`

All agents and skills live in the `tradify-agent-skills` repo. `setup.ps1` creates three symlinks:

| Symlink | Target |
|---|---|
| `$HOME\.claude\agents` | `tradify-agent-skills\agents\` (all 8 tech-lead `.md` files) |
| `$HOME\.claude\skills` | `tradify-agent-skills\` (skills live at repo top level) |
| `tradify\.claude\agents\*.md` | repo-level agents from `tradify-frontend-platform\.claude\agents\*.md` |

**Prerequisite:** enable Developer Mode (Settings → Privacy & security → For developers) **or** run PowerShell as Administrator. Without one, `New-Item -ItemType SymbolicLink` fails with "access denied".

```powershell
cd $HOME\development\tradify
git clone git@github.com:TradifyHQ/tradify-agent-skills.git
cd tradify-agent-skills
.\setup.ps1                  # both global + project
# or scope it:
.\setup.ps1 -Global          # only $HOME\.claude
.\setup.ps1 -Project         # only parent .claude
```

If you happen to be inside WSL2 instead, run `./setup.sh` — same effect.

Verify:

```powershell
Get-Item $HOME\.claude\agents, $HOME\.claude\skills | Select Name, Target
# both should resolve to the tradify-agent-skills repo
```

> **What setup.ps1 does NOT do:** it does not create or touch `CLAUDE.md`. See §2 for CLAUDE.md options (copy curated file, or run `/init` in-session).

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

Already in `tradify-frontend-platform\.claude\agents\` after `git clone`. Includes `bundle-performance-engineer`, `code-reviewer`, `devx-engineer`, `docs-engineer`, `frontend-architect-ui`, `frontend-context-architect`, `platform-engineer`.

Updating any agent: edit in `tradify-agent-skills`, commit, push. Symlinks pick up the change everywhere immediately.

**Workflow rule:** never implement directly. Delegate to the specialist agent for the domain.

---

## 8. Hooks (`%USERPROFILE%\.claude\settings.json`)

After installing caveman + rtk + icm + agent-deck:

```jsonc
{
  "hooks": {
    "SessionStart": [
      { "type": "command", "command": "node %USERPROFILE%\\.claude\\hooks\\caveman-activate.js" },
      { "type": "command", "command": "icm hook start" },
      { "type": "command", "command": "agent-deck hook-handler SessionStart" }
    ],
    "SessionEnd":   [{ "type": "command", "command": "icm hook end" },
                     { "type": "command", "command": "agent-deck hook-handler SessionEnd" }],
    "UserPromptSubmit": [
      { "type": "command", "command": "node %USERPROFILE%\\.claude\\hooks\\caveman-mode-tracker.js" },
      { "type": "command", "command": "icm hook prompt" },
      { "type": "command", "command": "agent-deck hook-handler UserPromptSubmit" }
    ],
    "PreToolUse": [
      { "matcher": "Bash", "hooks": [{ "type": "command", "command": "rtk hook claude" }] }
    ],
    "PreCompact":        [{ "type": "command", "command": "icm hook compact" }],
    "Stop":              [{ "type": "command", "command": "agent-deck hook-handler Stop" }],
    "Notification":      [{ "type": "command", "command": "agent-deck hook-handler Notification" }],
    "PermissionRequest": [{ "type": "command", "command": "agent-deck hook-handler PermissionRequest" }]
  }
}
```

**Windows hook gotchas:**

- Use **full paths** to executables if PATH inheritance is flaky (`C:\Users\<you>\AppData\Local\...\rtk.exe`).
- Backslashes in JSON must be escaped: `\\`.
- If hook commands silently fail, run Claude from PowerShell to see stderr.

### Caveman mode

```
/caveman lite     # mild
/caveman full     # JC's default
/caveman ultra    # max compression
```

Disable: say "stop caveman" or "normal mode" in any session.

---

## 9. Obsidian Vault — PARA layout + `plans\` symlink

### PARA structure

PARA (Tiago Forte): **P**rojects · **A**reas · **R**esources · **A**rchives. JC uses a Johnny-Decimal-flavoured PARA:

```powershell
mkdir "$HOME\Documents\Obsidian Vault"
cd "$HOME\Documents\Obsidian Vault"

# Create the folder tree:
"20-meetings",
"30-projects\react-migration\plans",
"40-hiring\scorecards",
"50-references",
"60-archive",
"99-MISC",
"_templates" |
  ForEach-Object { New-Item -ItemType Directory -Path $_ -Force | Out-Null }
```

| Folder | Maps to | What lives here |
|---|---|---|
| `20-meetings\` | Projects/Resources hybrid | Standups, steering committee, 1:1s |
| `30-projects\` | **Projects** | Active work: react-migration, TRA-XXXXX plans |
| `40-hiring\` | **Areas** | Ongoing area of responsibility (scorecards, candidates) |
| `50-references\` | **Resources** | Permanent reference docs (this playbook lives here) |
| `60-archive\` | **Archives** | Completed projects, retired notes |
| `99-MISC\` | inbox | Capture-then-sort |
| `_templates\` | — | Templated notes (sprint review, ADR, PR template) |

Open the vault in Obsidian → Settings → Files & links → enable "Default location for new notes: same folder as current file".

### `plans\` symlink — commit plans with code

Enable Developer Mode first (Settings → Privacy & security → For developers), THEN:

```powershell
cd $HOME\development\tradify\tradify-frontend-platform
New-Item -ItemType SymbolicLink `
  -Path plans `
  -Target "$HOME\Documents\Obsidian Vault\30-projects\react-migration\plans"

# Verify:
Get-Item plans | Select-Object Name, Target
```

If Developer Mode is off, you'll get an elevation prompt — accept or enable Dev Mode first.

### Don't use Obsidian?

Plans are just `.md` files. Point the symlink at any folder you maintain in any editor: VS Code, iA Writer, Typora, Logseq, plain text. The repo doesn't care.

---

## 10. Conventions stored as memory

Memory dir on Windows:

```powershell
$HOME\.claude\projects\-Users-<your-windows-username>-development-tradify\memory\
```

The path-as-dashed-folder-name encoding mirrors macOS. Match your real home dir exactly.

Same "always / never" rules apply — see the macOS handout §11 or the playbook §9. Easiest add: say "remember that X" in any session.

---

## 11. Pre-push agent-review hook (TRA-23186)

Husky scripts are POSIX shell — **do NOT push from cmd.exe**. Use one of:

- **Git Bash** (ships with Git for Windows)
- **WSL2 Ubuntu**
- **PowerShell** with Git Bash on PATH (works because husky shebangs to `sh`)

```bash
# In Git Bash (the platform repo uses bun):
cd ~/development/tradify/tradify-frontend-platform
bun install                          # husky auto-installs into .husky/_/

git push                             # tests + coverage only
git push -o agent-review             # also: parallel Claude agent review
git agent-push                       # alias for the above
SKIP_PRE_PUSH_TESTS=1 git push       # emergency hotfix
```

If `claude` CLI isn't on Git Bash's PATH, the hook warns and lets the push through.

To make `claude` reachable from Git Bash, add this to `~/.bashrc`:

```bash
export PATH="$PATH:/c/Users/<you>/AppData/Roaming/npm"
```

---

## 12. Verify your setup

```powershell
claude --version
icm recall "test"
rtk gain
agent-deck --version
Get-Item $HOME\.claude\agents | Select Name, Target
cd $HOME\development\tradify; claude    # cross-repo brain loads
```

Inside `tradify-frontend-platform\` only: `bun --version` should also work (that repo's package manager).

Inside Claude session: `/plugin` lists installed plugins. `/caveman full` activates compressed mode.

---

## 13. Day-1 onboarding checklist

- [ ] Enable Windows Developer Mode (for symlinks)
- [ ] Install Claude Code + icm + rtk + agent-deck
- [ ] Clone all 4 Tradify repos + `tradify-agent-skills` under `$HOME\development\tradify\`
- [ ] Drop in parent `CLAUDE.md` + `AGENTS.md`
- [ ] Run `tradify-agent-skills\setup.ps1` (symlinks all agents/skills globally + project)
- [ ] Install 8 high-value plugins
- [ ] Authenticate Atlassian + Figma + Slack + Chrome DevTools MCPs
- [ ] Symlink `plans\` to Obsidian vault (if applicable)
- [ ] Run `/caveman full`
- [ ] On your first PR, use `TRA-XXXXX - feature/bucket/name` title
- [ ] Add one `icm store` entry by EOW1
- [ ] Ping JC on Slack with anything to formalize as a feedback memory

---

## 14. Windows-specific things to skip

- Don't push from `cmd.exe` — POSIX hooks fail
- Don't run bun scripts from inside Cursor's integrated PowerShell if it doesn't inherit your PATH — open Windows Terminal instead
- Don't capitalize `.claude` or `CLAUDE.md` differently from JC's docs — path matching breaks
- Don't use `--force-push`; always `--force-with-lease`
- Don't `git stash` when a rebase is what you want

---

## 15. Trouble?

| Symptom | Fix |
|---|---|
| `claude` not auth'd | Re-run `claude` to refresh browser flow |
| ICM "command not found" | Re-add Cargo bin (`$HOME\.cargo\bin`) to PATH |
| rtk filtering breaks a command | `rtk proxy <cmd>` runs raw |
| Husky hook fails: "sh: not found" | Push from Git Bash, not cmd.exe |
| Symlink creation needs admin | Enable Developer Mode |
| MCP auth loops | Revoke + re-authenticate from third-party (Atlassian/Slack/Figma) |
| Hooks silently fail | Start Claude from PowerShell to see stderr |
| Path with spaces breaks command | Quote both halves of the path in PowerShell |

---

> "There is a difference between knowing the path and walking the path." — Morpheus

Slack: `#frontend` · JC reconciles new conventions at the next steering committee.
