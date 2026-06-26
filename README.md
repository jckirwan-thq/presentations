# Tradify Presentations

Reusable home for talks and decks by the Tradify Frontend team. Each presentation lives in its own subfolder. Deployed via GitHub Pages — push to `main` and the URL refreshes automatically.

**Live root:** https://tradifyhq.github.io/presentations/

## Presentations

| Slug | Title | Author | Live URL |
|---|---|---|---|
| [`claude-code-playbook/`](./claude-code-playbook/) | Welcome to the Real World — JC's Agentic Workflow | Jean-Claude Kirwan | https://tradifyhq.github.io/presentations/claude-code-playbook/ |
| [`productivity-analysis/`](./productivity-analysis/) | Captain's Log — Productivity Analysis (YTD 2026) | Jean-Claude Kirwan | https://tradifyhq.github.io/presentations/productivity-analysis/ |
| [`funnel-improvements/`](./funnel-improvements/) | Sharpening the Saw — Crew Funnel Thin-Mode (before & after) | Jean-Claude Kirwan | https://tradifyhq.github.io/presentations/funnel-improvements/ |
| [`automated-fix-versions/`](./automated-fix-versions/) | Automated Fix Versions — Sisyphus Gets a Day Off | Jean-Claude Kirwan | https://jckirwan-thq.github.io/presentations/automated-fix-versions/ |

## Adding a new presentation

1. Create a folder: `mkdir my-talk-slug`
2. Drop in an `index.html` (use reveal.js, marp, slidev — your call) plus any handout markdown
3. Add a row to the table above
4. Commit + push — Pages picks it up

## Conventions

- Folders named `kebab-case-slug/` — that becomes the URL path
- Each folder must have an `index.html` so the slug URL works without a trailing filename
- Handouts as `.md` siblings (e.g. `handout-mac.md`, `handout-windows.md`) — link from the deck
- No build step — everything self-contained or CDN-loaded
- Don't commit secrets / credentials / internal customer data — this repo is org-internal but assume anyone in TradifyHQ can read

## Local preview

```bash
cd presentations
# Any static server. Pick one:
python3 -m http.server 8000
# or:
npx serve .
```

Open http://localhost:8000/ to browse, http://localhost:8000/claude-code-playbook/ for the deck.
