# Futayomi roadmap

Futayomi is a fork of [Mihon](https://github.com/mihonapp/mihon) with optional real-time sync
between devices (phone + tablet), and later a built-in anime watcher.

## Locked decisions (approved by the user, 2026-10-28)

| Topic | Decision |
|---|---|
| Base | Fork of Mihon (manga). Upstream merged about once a month: `git fetch upstream && git merge upstream/main` |
| App id | `app.futayomi` (installs next to Mihon) |
| Sync backend | Firebase Firestore, Spark (free) plan, local-first design |
| Sync is optional | App works without any Firebase. Each user creates their own Firebase project |
| Firebase config | Entered at runtime in Settings (Project ID, API key, App ID). No `google-services.json` in the repo |
| Sign-in | Firebase Auth, email + password |
| Privacy | Payloads encrypted on the device with a user passphrase. Rules allow only the owner's uid |
| Repo | Public, `github.com/nideta231/futayomi` |
| Anime | Same app (one app), built from scratch later. Must not change how manga sync works |

## Phase 1: fork and rebrand (in progress)
- [x] Fork, `upstream` remote, rebrand (name, app id, launcher icon, updater repo)
- [ ] Debug build installed on the tablet

## Phase 2: sync module `:sync` (proposed design)
- Outbox table of change events, flushed immediately (page progress debounced ~3 s)
- Firestore layout: `users/{uid}/manga/{hash(source+url)}` (one doc per manga: library flag,
  categories, chapters read/bookmark/last page), `users/{uid}/meta/categories`
- Every doc carries `kind: "manga"` so the future anime part uses separate docs (`kind: "anime"`)
- Listener on docs with `updated_at > cursor` while the app is open; catch-up on app start
- Conflicts: read flag sticky unless explicitly unmarked, last write wins for page/categories,
  soft delete only (`deleted_at`), never delete by absence
- Quota error: stay local, show "sync paused"
- Hooks in Mihon code kept minimal (chapter, history, library, category writes, app start)

## Phase 3: anime (later, not designed yet)

## Before publishing a release
- [ ] Own release signing key (kept outside the repo, backed up off this PC)
- [ ] `docs/SYNC_SETUP.md` + `firestore.rules` template
- [ ] Credit Mihon in README, keep Apache-2.0 LICENSE
