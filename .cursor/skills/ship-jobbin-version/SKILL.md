---
name: ship-jobbin-version
description: Runs Jobbin's full release gate in order (regression, version sync, commit, push, deploy verify, tag, handover reminders). Use when the user says /ship, or asks to release, ship, or deploy a Jobbin version.
---

# Ship Jobbin Version

Run every step below **in order**. Explain each step in plain English before doing it. Stop immediately on failure — do not skip ahead.

## Rules

- One live index.html only. Never create duplicate/versioned files.
- Never mark a step done you haven't actually verified. If something fails, stop and tell me in plain English.
- I'm a coding novice: explain each step simply as you go.
- Do not change git config. If commit author email is wrong, stop and tell the user how to fix it locally.
- Never reference or rely on line numbers when editing index.html; match unique strings and re-read the changed section after each edit.

## Before you start

Confirm with the user (if not already given):

- Target version (e.g. `31.5` → tag `v31.5`)
- ClickUp **parent milestone** task id for the commit message (format `#869xxxxxx`, not a subtask)

## Step 1 — Regression check

Open the app (local `index.html` via browser automation, or the live site only if local changes are already committed — prefer local for pre-ship).

**Views:** Click every sidebar `data-view`: dashboard, overview, tasks, clients, internal, scps. Confirm each view renders without errors.

**Core flows:** Exercise representative add, edit, and delete flows on the main entity types (jobs/board, tasks, clients, internal contacts, supply chain as applicable). Use test data; avoid destructive actions on irreplaceable production data if testing live.

**Themes:** Toggle light and dark mode (`data-theme` / theme control). Re-spot-check key views in both themes.

Report anything broken and **STOP** if so. Do not proceed to commit until regression passes.

Detailed checklist: [regression-checklist.md](regression-checklist.md)

## Step 2 — Version number consistency

Update the version everywhere it belongs in `index.html`. Find each occurrence by **unique string** search; do not touch unrelated matches (SVG path coordinates, `DB_VERSION`, `CURRENT_SCHEMA_VERSION`, IndexedDB keys, etc.).

After edits, grep confirms all display/export version strings match. See [version-strings.md](version-strings.md) for the usual locations (re-verify with search — the file may have changed).

## Step 3 — Stage and commit

1. `git status` and `git diff` — only intended release changes staged.
2. Stage `index.html` (and any other intentional release files).
3. Commit message must reference the ClickUp parent milestone as `#<taskid>` (never `CU-` id, never a subtask id).
4. After commit, verify author email: `git log -1 --format='%ae'` must be the GitHub noreply address (`*@users.noreply.github.com`), not a personal email.

Example message shape:

```text
feat: short release summary #869e3bz4z
```

## Step 4 — Push to origin main

```bash
git push origin main
```

If push fails, stop and report the error in plain English.

## Step 5 — Confirm Cloudflare Pages deploy

After push, confirm production updated:

1. Wait for deploy (Cloudflare dashboard or `gh`/Pages status if available).
2. Fetch the live site at **jobbin.pages.dev** and the **custom domain** (if configured).
3. Confirm the visible version matches the release (title, sidebar footer, header).

If the old version still shows after a reasonable wait, stop and report — do not tag until live matches.

## Step 6 — Git tag

Create and push a tag matching the version:

```bash
git tag v<major>.<minor>
git push origin v<major>.<minor>
```

Example: version `31.4` → tag `v31.4`. Tag only after Step 5 passes.

## Step 7 — Handover reminders

Tell the user explicitly they still need to:

1. **Publish the GitHub Release** for the new tag.
2. **Update the ClickUp changelog** for the milestone.
3. **Write the end-of-version handover note** and a **copyable next-chat prompt** for the next version.

Do not claim the release is fully “done” until the user completes these manual follow-ups.

## Progress tracking

Copy and update while running:

```text
Ship progress:
- [ ] Step 1 Regression (light + dark)
- [ ] Step 2 Version strings synced
- [ ] Step 3 Committed (#taskid, noreply author)
- [ ] Step 4 Pushed main
- [ ] Step 5 Live site shows new version
- [ ] Step 6 Tag pushed
- [ ] Step 7 User reminded (GitHub Release, ClickUp, handover)
```

Only check a box after that step is verified.
