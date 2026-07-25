# Jobbin regression checklist

Use browser automation (Cursor browser MCP) or manual testing. Record pass/fail per item.

## Navigation (each view must load)

Sidebar buttons (`data-view`):

- [ ] dashboard (Overview)
- [ ] overview (Job Board)
- [ ] tasks
- [ ] clients
- [ ] internal
- [ ] scps (Supply Chain)

## Theme

- [ ] Light mode — spot-check dashboard, one directory view, tasks
- [ ] Dark mode — same spot-checks

## Core flows (smoke — add / edit / delete)

Pick safe test records; undo or delete test data when done.

- [ ] Job board: open a job, edit a field, save; add and remove a test job if supported
- [ ] Tasks: toggle or edit a task; add/delete a test task if supported
- [ ] Clients: open client detail; add/edit/delete smoke test
- [ ] Internal: contact add/edit/delete smoke test
- [ ] Supply Chain: list loads; add/edit/delete or filter smoke test

## Console / obvious breakage

- [ ] No uncaught errors in browser console during the walkthrough
- [ ] No blank main panel after navigation

If any item fails, **stop the ship** and report what broke, which view, and light vs dark.
