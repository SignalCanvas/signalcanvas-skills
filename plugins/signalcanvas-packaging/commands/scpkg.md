---
description: Package a new SignalCanvas release — picks branch, bumps version, updates changelog and BETA-README, then builds the .pkg installer
---

You are packaging a new SignalCanvas release. Run from the FrontendV1 repo root — all paths below are relative.

## Step 0 — Pick the branch to build from

Run:

```bash
git branch --show-current
git branch | head -15
```

Show the user the current branch and the list of local branches. Ask: "Build from which branch? (default = current)"

If the user picks a different branch:

1. Check the working tree is clean:

   ```bash
   git status --porcelain
   ```

   If output is non-empty, **abort** and tell the user to commit or stash first. Do not stash automatically — branch switches across stashes lose work.

2. Switch to the chosen branch:

   ```bash
   git checkout <branch>
   ```

3. Do **not** create new branches. SignalCanvas project policy: no new branches. If the user names a branch that doesn't exist, abort and ask which existing branch they meant.

4. After the build completes, do **not** auto-restore to the original branch. The version bump should live on the build branch where the user can commit it.

## Step 1 — Detect current version

Read the `"version"` field from `tauri/tauri.conf.json`. This is the source of truth and should match `package.json` and `tauri/Cargo.toml`.

## Step 2 — Determine new version

If $ARGUMENTS contains a version number (e.g. `0.2.1`), use it. Otherwise auto-increment the patch number (0.2.0 → 0.2.1).

Tell the user: "Current version: X.X.X → New version: X.X.X" and ask them to confirm before continuing.

## Step 3 — Derive changelog entries from git

Run the following to find commits since the last version-bump commit:

```bash
git log --oneline $(git log --oneline | grep -m1 "bump version\|chore: bump\|version to $(node -p "require('./package.json').version")" | awk '{print $1}')..HEAD
```

If that produces no useful output, fall back to:

```bash
git log --oneline -20
```

Read the commit messages and also scan the actual diffs for user-facing changes:

```bash
git diff HEAD~10..HEAD -- src/ tauri/src/ 2>/dev/null | head -300
```

From the commits and diffs, synthesise a changelog in the existing style — group into Added / Fixed / Improved / Changed. Use bold feature names followed by an em dash and a one-sentence description, matching the style already in `releases/CHANGELOG.md`.

Present your proposed changelog to the user and ask: "Does this look right? (edit or confirm)"

Wait for their response before proceeding.

## Step 4 — Update version in all manifests

Edit these three files, replacing the old version string with the new one:

- `package.json` — top-level `"version"` field
- `tauri/tauri.conf.json` — top-level `"version"` field
- `tauri/Cargo.toml` — `[package]` `version =` field

The home-screen version label updates automatically — it's wired through Vite `define: __APP_VERSION__` from `package.json`, so no UI files need editing.

## Step 5 — Update releases/CHANGELOG.md

Insert a new entry at the top of `releases/CHANGELOG.md`, immediately after the header block and first `---` divider. Use today's date (YYYY-MM-DD format). Format:

```
## [NEW_VERSION] — YYYY-MM-DD

### Added
- **Feature** — description

### Fixed
- **Bug** — description

---
```

Only include sections (Added / Fixed / Improved / Changed / Removed) that have entries.

## Step 6 — Create releases/NEW_VERSION/BETA-README.md

Per-version BETA-README lives at `releases/<NEW_VERSION>/BETA-README.md`. Seed it from the previous version's README so testers downloading older builds still see correct instructions:

```bash
mkdir -p releases/NEW_VERSION
cp releases/OLD_VERSION/BETA-README.md releases/NEW_VERSION/BETA-README.md
```

Then edit `releases/NEW_VERSION/BETA-README.md`:
- Update the title: `# SignalCanvas Beta — vNEW_VERSION`
- Update the installer filename in the install table: `SignalCanvas-Installer-NEW_VERSION.pkg`
- Update the "version label confirmation" line: `vNEW_VERSION · local workspace`
- Update the "Known limitations" section if anything changed (ask the user if unsure)
- Update the "What works in this build" section if anything changed (ask the user if unsure)

Do not edit the previous version's BETA-README — it stays frozen for testers on that build.

## Step 7 — Confirm and build

Show the user a summary of all files changed (the 3 manifest bumps, CHANGELOG entry, new BETA-README), then ask: "Ready to build? This will take 3–5 minutes."

Once confirmed, run the build:

```bash
./scripts/build-desktop.sh
```

`build-desktop.sh` reads the version from `package.json`, runs `npm run tauri:build`, then invokes `scripts/build-pkg.sh` to wrap the `.app` into a `.pkg` installer.

## Step 8 — Report completion

When the build finishes, tell the user:
- New version shipped: X.X.X
- Installer: `releases/X.X.X/SignalCanvas-Installer-X.X.X.pkg`
- BETA-README: `releases/X.X.X/BETA-README.md`
- Branch: <branch name from Step 0>

Remind them to:
1. Commit the version bump (`package.json`, `tauri/tauri.conf.json`, `tauri/Cargo.toml`), `releases/CHANGELOG.md`, and the new `releases/X.X.X/BETA-README.md` — but **only after they explicitly ask** for the commit (SignalCanvas project rule: never auto-commit).
2. Distribute the `.pkg` to beta testers along with the BETA-README install instructions.
