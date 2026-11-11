# releases

Signed release manifests and build files for vukyn desktop apps. **Build files only** — no source code, no secrets,
no private keys.

## Rules for every app (humans and agents)

Read this before planning a release flow for any app that publishes here. The reference implementation is
remuxer's `scripts/release.sh` + `scripts/release-prune.sh` (`make release` / `make release-prune`).

### Layout

- One folder per app: `<app>/latest.json`, the app's signed update manifest. `<app>` is a lowercase identifier.
- One GitHub release per version: tag `<app>-v<MAJOR.MINOR.PATCH>`, title `<app> <MAJOR.MINOR.PATCH>`, the build
  files as assets.
- Strict semver only (digits, no leading zeros, no pre-release or build part). A new version must be strictly newer
  than the one the app's `latest.json` names.
- The source repo tags the same version `v<MAJOR.MINOR.PATCH>`.

### Publishing order

1. Preflight: a clean tree on the source repo's `main`, pushed; the tag absent here; the version newer than the
   published one.
2. Build, then sign the manifest with the app's own ed25519 key. The private key never enters any repository.
3. Create the release with its assets.
4. Re-download every asset the manifest names from its public URL and compare its SHA-256 with the signed one.
5. Publish `<app>/latest.json` **last**, commit message `<app> <version>`. Never edit a manifest by hand.
6. Tag `v<version>` on the source repo and push it.
7. Prune (below).

### Retention — keep only the newest

After each release, every app keeps only its **3 newest releases** (by semver, never by date; configurable per run,
at least 1). For every version strictly **older** than the oldest kept one, delete:

- the release and its tag here (`gh release delete <app>-v<V> --repo vukyn/releases --yes --cleanup-tag`);
- a leftover `<app>-v<V>` tag here that has no release;
- the source repo's `v<V>` tag, on its origin and locally.

Never delete:

- the version the app's `latest.json` points at. If the manifest cannot be read or verified, or points outside the
  kept set, stop and delete nothing;
- a tag of a newer version;
- any tag that is not exactly `<app>-v<MAJOR.MINOR.PATCH>` / `v<MAJOR.MINOR.PATCH>`, or another app's tags. This
  repository is shared, so a prune only ever touches its own app's prefix.

Installed apps download only the version their manifest names, so an older release is never needed by an update.

### Safety

- Publishing and pruning are **public and irreversible**. Every release and prune script ships a dry-run mode that
  lists what it would do and touches nothing.
- An agent may build and run the dry runs. It never runs the real publish or prune: the owner does.
