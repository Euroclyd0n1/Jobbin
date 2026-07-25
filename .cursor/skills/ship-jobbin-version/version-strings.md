# Jobbin version strings (index.html)

Re-run search before each release — this list is a guide, not a substitute for grep.

## Update these (product version)

| Unique anchor | Example | Notes |
|---------------|---------|--------|
| `<title>Jobbin · v` | `Jobbin · v31.4</title>` | Appears twice in file (minified head + main head) |
| `Jobbin — v` | `Jobbin — v31.4` | Visible app title |
| `Saved on this device · v` | `Saved on this device · v31.4` | Sidebar footer |
| `v:'` inside backup payload | `v:'31.4'` | Export/backup JSON metadata (no `v` prefix in string) |
| `/* v` release comment | `/* v31.4: auto-backup` | Keep comment prefix in sync |

All product version numbers must match (e.g. `31.4` in UI and `'31.4'` in JSON).

## Do not change (not product version)

- `DB_VERSION`, `CURRENT_SCHEMA_VERSION`, `schemaVersion` — data schema, not app release
- `KEY='ewt.v1'`, `FILE_DB='ewt.files.v1'` — storage keys
- SVG `viewBox`, path `d=` coordinates containing decimal numbers
- Unrelated numeric constants in JS

## Verification command

From repo root:

```bash
rg "Jobbin · v|Jobbin — v|Saved on this device · v|v:'[0-9]|/\* v[0-9]" index.html
```

Every hit should show the **same** new version.
