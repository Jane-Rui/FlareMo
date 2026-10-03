# Jane-Rui/FlareMo deployment operations

Updated: 2026-10-03

## Upstream updates

`Prepare FlareMo update` checks the latest stable release daily. It applies the binary/full-index difference between the installed and target releases with `git apply --3way --index`. It preserves all local workflows, `.gitignore` and `wrangler.jsonc`. Existing open update PRs are preserved rather than force-overwritten. A conflict fails without changing main.

`Sync Upstream` is disabled: it previously duplicated the update task and used history-dependent merges. Changing upstream workflows or protected deployment configuration requires a separately reviewed change.

## Production release

An upgrade PR does **not** automatically deploy. Review the upstream notes, cumulative migrations, configuration-template changes, and resource identities first.

`Deploy to Cloudflare Workers` is manual. Start with `dry_run=true`: dependency installation, frontend build, existing-resource checks and Worker dry-run. Resources are never automatically replaced. For production, secure a verified D1 backup, set `dry_run=false` and `backup_confirmed=BACKUP_CONFIRMED`. The task applies pending D1 migrations before publishing the Worker and checks the public origin.

The deployed database is the existing `flaremo` database; attachments and vector indexes keep their existing identities. Secrets remain in GitHub Secrets and the live Worker secret store. Never place keys, sessions, credentials or database exports in Git.

## Important v0.20.0 to v0.22.0 migration boundary

Cumulative migrations 0019–0033 include migration 0020, which copies legacy roles into team memberships and **drops users.role**. Consequently, deploying the old v0.20.0 Worker alone after migration is not a safe rollback. Keep a pre-migration Time Travel bookmark plus a verified local data/schema backup. Recovery to old code requires database recovery as well and review of writes that occurred after the backup; do not blindly restore a database while users are writing.

A D1 whole-database export fails when FTS5 virtual tables are present. Backups must cover every non-derived business table and schema; FTS indexes can be rebuilt from the authoritative tables. Test restores and migrations on a local copy before publication. Backup files are private and must not be attached to an issue or PR.

## Acceptance

Do not equate an already-installed skip, GitHub success status, or homepage HTTP200 with a working upgrade. Verify an actual version-to-version update PR; build and dry-run; check active Cloudflare version, applied migrations, preserved memo content, authenticated reads/search, and a private temporary write/read/delete. A browser password login requires the owner or an existing authenticated browser session and must not be claimed when not tested.
