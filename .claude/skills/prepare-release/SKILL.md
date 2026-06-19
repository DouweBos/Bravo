---
name: prepare-release
description: Use this skill when the user asks to "prepare a release", "update the changelog", "bump the version", "prepare release notes", or "cut a release". Automates changelog generation from git commits since the last tag and version bumping across gradle.properties files.
version: 1.0.0
---

# Prepare Release Skill

Automates the release-prep workflow: a CHANGELOG.md entry plus version bumps.

## Versioning scheme

Bravo versions are **`major.minor.patch.build`**:

- **`major.minor.patch`** tracks upstream Maestro — `CLI_VERSION` in
  `maestro-cli/gradle.properties` and `VERSION_NAME` in `gradle.properties`. These
  are overwritten by upstream on rebase; we follow them.
- **`build`** is the fork-owned segment — `BRAVO_BUILD`. It is **auto-incremented at
  release time** by the `publish-cli` workflow (it finds the highest existing
  `cli-<CLI_VERSION>.<N>` release and uses `N+1`, or `0` for a new Maestro version),
  so fork-only fixes ship without drifting from Maestro's version. You do **not**
  hand-edit it; the value in `maestro-cli/gradle.properties` is only a local-build
  fallback.

The composed `major.minor.patch.build` is assembled in `maestro-cli/build.gradle.kts`
(`FULL_CLI_VERSION`) and used for `version.properties`, the JReleaser tag
(`cli-X.Y.Z.B`), and the release name. The CHANGELOG and its consistency test stay
keyed on the 3-part `CLI_VERSION` (the build segment is fork metadata, not a
Maestro changelog version).

## Steps

### 1. Determine the release type and version

Decide which kind of release this is:

- **Fork-only release** — a fix to Bravo's own additions on the *same* Maestro base.
  Nothing to bump: leave `CLI_VERSION`/`VERSION_NAME` as-is. `publish-cli` will pick
  the next `BRAVO_BUILD` automatically when you dispatch it. This skill's only job
  here is an optional CHANGELOG bullet (step 3); steps 4 is a no-op.
- **Upstream-tracking release** — `major.minor.patch` is moving (typically right
  after an upstream rebase). Set `CLI_VERSION`/`VERSION_NAME` to the new Maestro
  version (step 4); `BRAVO_BUILD` resets to `0` on its own, since there are no
  `cli-<new version>.<N>` releases yet.

If the user hasn't said which, use `AskUserQuestion` to confirm the type and the
target `major.minor.patch` (no `v` prefix).

### 2. Find commits since the last tag

Run these two commands:

```bash
git describe --tags --abbrev=0
```

Then use the resulting tag (e.g. `v2.5.0`) to list commits:

```bash
git log <tag>..HEAD --oneline
```

### 3. Update CHANGELOG.md

File: `CHANGELOG.md`. CHANGELOG headers are the **3-part** `major.minor.patch`.

- **Upstream-tracking release**: insert a new section **between** the `## Unreleased`
  line and the first existing version header, headed by the new `major.minor.patch`.
- **Fork-only release** (`BRAVO_BUILD` bump, same `major.minor.patch`): the section
  for the current `major.minor.patch` already exists — append the fork fix as a new
  bullet there rather than adding a new header. (The consistency test only requires a
  non-empty entry for `CLI_VERSION`.)

Section format:

```markdown
## <major.minor.patch>

- <entry 1>
- <entry 2>
...
```

**Editorial rules for commit → entry conversion:**
- Each commit becomes one `- ` bullet point.
- Strip conventional-commit prefixes (`fix:`, `feat:`, `fix(scope):`, etc.) and capitalise the first word.
- Remove PR number suffixes like `(#1234)` only if the commit message is already clear without them; otherwise keep them.
- Rewrite for clarity and consistency with the existing CHANGELOG tone (sentence case, imperative or noun phrases).
- Do **not** add a "Thanks to" contributors line — that is added manually.

### 4. Update version numbers (upstream-tracking release only)

For a **fork-only release**, skip this step — leave the version files untouched.

For an **upstream-tracking release**, set the new 3-part Maestro version in both
files. Leave `BRAVO_BUILD` alone (it is auto-incremented at release time):

**`gradle.properties`** — `VERSION_NAME` (3-part Maestro version):
```
VERSION_NAME=<major.minor.patch>
```

**`maestro-cli/gradle.properties`** — `CLI_VERSION` (3-part):
```
CLI_VERSION=<major.minor.patch>
```

### 5. Run ChangeLogUtilsTest

```bash
./gradlew :maestro-cli:test --tests "maestro.cli.util.ChangeLogUtilsTest"
```

The key test (`test format last version`) reads the generated version, takes its 3-part base (`major.minor.patch`), and asserts the CHANGELOG contains a non-empty entry for it. So the CHANGELOG must have a populated section for the current `CLI_VERSION` — the `BRAVO_BUILD` segment is ignored by this check.

Report the test result. If it fails, diagnose and fix before finishing.
