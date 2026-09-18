---
name: hackspain-cli
description: Reference and guardrails for the `hackspain` CLI (HackSpain 2026 terminal companion) — auth, team, track, project/submit, milestones, stack, feed/post, watch/telemetry. Use whenever a task involves the HackSpain dashboard, checking team/project/track status, recording a milestone, posting to the feed, or submitting the project.
---

# hackspain CLI

Terminal companion for HackSpain 2026 (Madrid). Docs: https://hackspain.app/cli (behind a Vercel
bot checkpoint — `hackspain <cmd> --help` is the reliable source). Documented against **v0.5.1**;
if `hackspain --version` differs, re-check `--help` before trusting this file.

Binary: `~/.local/bin/hackspain`. Session is stored locally by `hackspain auth login`.

## Rules for agents

1. **Always pass `--json` when reading.** It gives machine-readable output on stdout and disables
   prompts: `hackspain --json team show`. Response envelope: `{"ok":true,"data":{...}}`.
   `--json` is a *global* option, so it goes before the subcommand.
2. **Read freely, write only when asked.** The commands in "Read-only" below are safe to run at any
   time. Everything in "Mutating" changes shared team state or is visible to every participant and
   the organisers — run those only on an explicit request from the user, and never add `-y/--yes`
   unless the user said to skip confirmation.
3. **`hackspain submit` without `--draft` is the final submission.** Default to `--draft`. Only do a
   real submit when the user explicitly says to submit.
4. **Never run `hackspain watch` on your own.** It is a long-running full-screen process that scans
   this machine for AI-harness usage and uploads events to the dashboard. It is the user's call to
   run it (in their own terminal). `hackspain telemetry stats` shows what it has recorded locally.
5. **Never print or commit the team join code** (`team show` / `team code` return it). This repo is
   public and pushes to it show up in the HackSpain feed. Same for teammates' emails.
6. **Auth is interactive.** If `auth status` shows `loggedIn:false`, ask the user to run
   `! hackspain auth login` (browser flow) — do not attempt the email-code flow yourself.
7. `hackspain update` replaces the binary; only run it if the user asks (`update --check` is safe).

## Read-only (safe)

| Command | What it returns |
| --- | --- |
| `hackspain --json auth status` | Logged-in email, gate state (`ready` = accepted and onboarded), token expiry |
| `hackspain` (no args) | "Where you stand" overview |
| `hackspain --json team show` | Team, members, linked repo(s), stack. **Contains the join code — do not echo it.** |
| `hackspain --json team list` | All teams with member counts, tracks, repos |
| `hackspain --json track list` | Tracks, `entered` flag per track, `submissionsOpen` |
| `hackspain --json project show` | Our project: name, description, status (`draft`/submitted), urls, perks |
| `hackspain --json project list` | Every named project |
| `hackspain --json perk list` | Partner perks and claim status (ids are used by `submit --perk`) |
| `hackspain --json milestone list [-a]` | Our milestones (`-a` = all teams) |
| `hackspain --json stack show` | Team stack (detected or hand-set) |
| `hackspain feed [-n 20] [--before <cursor\|ISO>] [--no-images]` | Feed posts + pushes from team repos |
| `hackspain --json profile show` | Own participant profile |
| `hackspain open --print [page]` | Prints a signed-in dashboard link instead of opening a browser. Treat the link as a credential. |
| `hackspain telemetry stats` | Local watcher spool totals by harness and model family |
| `hackspain update --check` | Whether a newer release exists |

## Mutating (only on explicit user request)

### Project and submission
```bash
# Save progress — safe to repeat, everything stays editable
hackspain submit --draft --name "<name>" --description "<>=10 chars>" \
  --repo https://github.com/<org>/<repo> --demo <url> --video <url> --track maisa

# FINAL submission — only when the user says so
hackspain submit --name ... --description ... --repo ... --track maisa
```
`--repo` must be a **public** GitHub URL. `--video` accepts YouTube, Loom or MP4. `--track` and
`--perk <id>` are repeatable. `-y` skips the final confirmation.

### Tracks
```bash
hackspain track register <slug...>     # slugs: maisa, happyrobot, prosper-ai, embat, theker
hackspain track unregister <slug...>
hackspain track move <from> <to>
```

### Milestones (feed the organisers' live insights)
```bash
hackspain milestone add firstCommit
hackspain milestone add firstBuild
hackspain milestone add firstDemo
hackspain milestone add custom -l "Batch 1 validated" [--at <ISO timestamp>]
```

### Stack and repo
```bash
hackspain team repo [urls...] [-y] [--clear]   # show or set the linked GitHub repo(s)
hackspain stack detect [-y]                    # read the stack from the linked repo(s)
hackspain stack set Python FastAPI ...         # replace the stack by hand
```

### Feed
```bash
hackspain post "short update" [-i image.png]   # jpg/png/webp/gif, up to 5 MB — public to all participants
```

### Team management (owner-sensitive — confirm every time)
`team create <name> [-m github:<login>|email:<addr>|twitter:<handle>]`, `team join <8-char code>`,
`team leave`, `team code --regenerate` (invalidates the current code), `team transfer [member]`,
`team dissolve`. The last four are hard to undo.

### Profile
`profile edit [--name --diet --diet-details --from]`, `profile notify <on|off>`,
`profile phone [number]`, `profile x [handle|--clear]`, `profile github [--unlink]`.

### Auth
`auth login` (browser) · `auth login -e <email> [--code <8 digits>]` (email code) · `auth logout`.

## Global options

| Option | Meaning |
| --- | --- |
| `--json` | Machine-readable stdout, no prompts |
| `--url <url>` | Dashboard URL. Resolution order: flag, `HACKSPAIN_APP_URL`, config, then `hackspain.app` |
| `-v, --version` | Print version |

## Our state (snapshot 2026-09-18 — re-check with the read-only commands)

- Logged in, gate `ready`. Team of 4 exists; this repo is the team's linked repo.
- Entered in track `maisa`. Project is an empty `draft`. `submissionsOpen` was `false`.
- Stack not set yet — run `hackspain stack detect` once there is code in the repo.

## Do not confuse the two submissions

`hackspain submit` is the **HackSpain event** submission (name, description, repo, demo, video).
The **Maisa track** has its own deliverable: a *separate* public GitHub repo whose root holds exactly
`outcomes.jsonl`, `outcomes_lote2.jsonl` and `albertitos_plan.pdf` (see `README.md`). Both are needed.
