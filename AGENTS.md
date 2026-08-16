# AGENTS.md

Standing rules for any AI coding agent working in this repo (Copilot Chat, Codex, Cline, Cursor, Ollama-served models). Read this before making any edit.

## What this project is

Jobbin is a **single-file HTML job/quote tracker**. Vanilla HTML/CSS/JS, no build step, no framework, no bundler. Everything lives in `index.html`.

- Hosted on Cloudflare Pages at https://jobbin.pages.dev
- Auto-deploys from branch `main` on push. There is no manual deploy step.
- `index.html` at the repo root is what gets served. Do not rename or relocate it.

## The one rule that matters most: anchored edits

The owner cannot see reliable line numbers and does not read code fluently. **Never locate an edit by line number.** Line counting produces miscounts and broken pastes.

Instead, for every edit:

1. Locate the change by a **unique Ctrl+F anchor string** that appears exactly once in the file.
2. Name the line **directly above** and **directly below** the anchor so the location can be confirmed by eye.
3. Give the **find (before)** text and the **replace (after)** text in **separate fenced code blocks**. Never inline in prose: inline text drags in stray whitespace and breaks the paste.
4. Say new content goes **"on a new line BELOW"** a landmark. Never say "after" (it reads as same-line).

## Scope discipline

- Change **only** what was asked. Do not reformat, reindent, tidy, rename or "improve" surrounding code.
- Do not reorder or touch any other `<script>` tag while editing one.
- Do not reflow whitespace across the file. A diff should be small enough to read in one screen.
- Each edit block must be **final for that section**. No add-now-remove-later churn. Combine multiple changes to one section into a single block; split only across genuinely different sections of the file.
- If a change would touch more than a couple of regions, stop and describe the plan first.

## Read the region, not the whole file

`index.html` is very large. Do not load the entire file into context for a small change. Find the anchor, read the surrounding region, edit that. This keeps token costs down and keeps the edit focused.

## Build up, not sideways (BUNS)

Build foundations before the things that mount into them. Do not build a second version of something that already exists.

Specifically:

- There is **one** per-person settings store. Never invent a second one.
- There is **one** settings surface. Never build a parallel settings panel.
- One live `index.html`. Version history lives in git commits and tags, **never** in duplicate files like `index_v32.html`.

## Storage and migrations

- Register and attachments live in **IndexedDB**, database `jbn.files.v1`.
- Renamed from `ewt.files.v1` on 16 Aug 2026 (the old Extra Works Tracker prefix). Do not reintroduce `ewt` anywhere.
- The `KEY` constant (`jbn.v1`) is a legacy `localStorage` read left over from pre-v31 builds. Nothing writes it, so it always returns null and boot falls through to IndexedDB. It is dead but harmless, and is slated for removal along with `lsGet` and `migrateLegacyAttachments`.
- A **schema-version constant + ordered migration runner** shipped in v31.4.

If a change reshapes persisted data you MUST: (1) write a new ordered migration step, and (2) increment the schema-version constant. Shipping a data-shape change without registering its upgrade step breaks existing data and Drive backups on load.

Note that the migration runner reshapes the data object **after** it has been loaded. It does not and cannot rename the database the object came out of. Renaming an IndexedDB database is a separate copy-then-delete job, not a schema step.

## Google Drive backup / OAuth

Drive backup requires a real `http(s)` origin listed in the OAuth client's **Authorised JavaScript origins**. `file://` and embedded preview iframes both fail. To test locally:

```
python3 -m http.server 8000
```

and add `http://localhost:8000` as an authorised origin.

The Drive backup filename is `jobbin-tracker.json`, independent of the IndexedDB database name.

Do not remove or relocate the Google OAuth script or the lucide icons path. Both have been broken before by well-meaning cleanup.

## Commit conventions

- Reference the ClickUp task in the commit message as `#<taskid>`, e.g. `#869ecvbcu`. Optionally `#<taskid>[status]` to also move its status.
- Do **not** use `CU-<id>` or a task URL. ClickUp does not parse those.
- Always reference the **parent milestone / version task**, never a subtask, so all commits for a version collect in one place.
- Author email is fixed to the GitHub noreply: `281914770+Euroclyd0n1@users.noreply.github.com`. This avoids GitHub's GH007 private-email push rejection. If GH007 appears, check `git config user.email`.
- Commit and tag before deploying.

## Naming

Task titles, commit subjects and user-facing labels use **plain English**. No jargon, no acronyms. Configuration detail belongs in the description, not the title.

## Never do these

- Never save Jobbin via a browser's "Save as Complete webpage". That wrapper strips the Google OAuth script, kills the lucide icons path and injects dead references. Only ever ship clean committed source.
- Never create duplicate versioned HTML files.
- Never blind-apply an edit without showing the diff first.
- Never widen scope beyond the request without asking.

## After a change

Re-read the file in the repo after the push to confirm the edit landed as intended. Do not assume the paste was clean.
