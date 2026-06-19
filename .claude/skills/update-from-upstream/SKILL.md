---
name: update-from-upstream
description: Use when asked to update/sync the Bravo fork with the latest Maestro, rebase on upstream, or "pull in upstream changes". Rebases Bravo's main onto upstream/main, walks conflicts, re-verifies the fork commit registry, and force-pushes the fork.
---

# Update From Upstream

## Overview

Bravo (see [`CLAUDE.md`](../../../CLAUDE.md)) is a minimal stack of feature commits on top of `upstream/main`. "Updating from upstream" means **rebasing that stack onto the latest Maestro** so we pick up upstream changes while keeping our features on top.

## When to Use

- User asks to sync/update Bravo with Maestro, "rebase on upstream", or pull in upstream changes.

## Pre-flight

- Working tree clean (`git status`).
- Remotes present: `origin` → `DouweBos/Bravo`, `upstream` → `mobile-dev-inc/Maestro` (`git remote -v`). Add `upstream` if missing (see `CLAUDE.md`).
- Snapshot the current tip so the rebase is recoverable:
  ```bash
  git branch -f bravo-backup main
  ```

## Procedure

1. **Fetch upstream:**
   ```bash
   git fetch upstream
   ```

2. **Inspect the current fork stack** (this is what gets replayed):
   ```bash
   git log --oneline upstream/main..main   # before fetch this is the OLD base; confirm the subjects
   git log --oneline main...upstream/main  # divergence overview
   ```
   Cross-check the subjects against the **Fork commit registry** in [`README.md`](../../../README.md).

3. **Rebase onto the new upstream:**
   ```bash
   git switch main
   git rebase upstream/main
   ```

4. **Resolve conflicts.** Common offenders:
   - **Checked-in driver binaries** — `maestro-ios-driver/src/main/resources/**/*.zip` and `*.xctestrun` (incl. the tvOS `driver-appletvSimulator/**`). Binary conflicts: keep our fork's version unless upstream rebuilt the driver, then prefer upstream and re-verify our tvOS additions still apply.
   - **`maestro-ios-xctest-runner/maestro-driver-ios.xcodeproj/project.pbxproj`** — frequently rewritten upstream; reapply our tvOS target/scheme entries by hand if needed.
   - **`README.md`** — keep the Bravo rebrand + registry; take upstream content elsewhere.
   - **`mapToSeleniumKey()`** in `WebDriver.kt` / `CdpWebDriver.kt` — keep our REMOTE_DPAD branches.

   After resolving each step: `git add <files> && git rebase --continue`. Abort with `git rebase --abort` (then `git reset --hard bravo-backup`) if it goes sideways.

5. **Re-verify the registry.** Every fork commit subject in `README.md`'s registry must still map 1:1 to a commit in `git log --oneline upstream/main..main`. If upstream merged one of our features (so the commit dropped out during rebase), remove that row from the registry and commit the README update (amend into the `docs: create Bravo` commit to keep the stack minimal).

6. **Build check** (fast sanity):
   ```bash
   ./gradlew :maestro-client:compileKotlin
   ```

7. **Push the updated fork** (rebase rewrites history, so force is required):
   ```bash
   git push --force-with-lease origin main
   ```

## Notes

- Only force-push `origin` after the rebase is clean and the registry is reconciled.
- Keep the stack minimal: prefer amending existing fork commits over adding new ones.
