# CLAUDE.md — Bravo

**Bravo** is Houwert's internal fork of [Maestro](https://github.com/mobile-dev-inc/Maestro). It exists so we can ship the features we need **ahead** of upstream accepting our PRs, and run a full multi-platform Maestro runner internally without waiting on the upstream merge queue.

This file is the entry point for anyone (human or agent) working in this repo. For the module-by-module map of the codebase, see [`AGENTS.md`](AGENTS.md).

## Remotes

Two remotes are configured:

| Remote     | URL                                          | Role                                                              |
|------------|----------------------------------------------|-------------------------------------------------------------------|
| `origin`   | `https://github.com/DouweBos/Bravo.git`      | Our fork. `main` here is the canonical Bravo branch.              |
| `upstream` | `https://github.com/mobile-dev-inc/Maestro.git` | The original Maestro repo. We rebase `main` on `upstream/main`. |

`upstream` is the "second origin" — point it at the original Maestro repo if it is ever missing:

```bash
git remote add upstream https://github.com/mobile-dev-inc/Maestro.git
```

## How the fork is structured

Bravo is **a minimal, linear stack of feature commits on top of `upstream/main`** — not a long-lived divergent branch. Keeping the stack small and each commit self-contained is what makes rebasing onto a fresh upstream tractable.

Rules of the road:

- **Keep the number of fork commits minimal.** Prefer **amending** an existing fork commit over adding a new one when a change belongs to an existing feature.
- **One feature per commit**, with a conventional-commit subject (`feat(...)`, `fix(...)`, `docs:`). When importing an upstream PR of ours, **squash** it into a single commit.
- **The commit registry lives in [`README.md`](README.md)** ("Fork commit registry"). It is keyed on **commit subject + purpose, not SHA** — every rebase on upstream rewrites the SHAs, but the subjects are stable. Update the registry whenever you add, amend, or drop a fork commit.

## Staying in sync with upstream

Use the [`update-from-upstream`](.claude/skills/update-from-upstream/SKILL.md) skill to rebase Bravo on the latest Maestro. In short: `git fetch upstream`, rebase `main` onto `upstream/main`, resolve conflicts (the checked-in iOS/tvOS driver binaries and `maestro-driver-ios.xcodeproj/project.pbxproj` are the usual offenders), re-check the registry, then `git push --force-with-lease origin main`.

## Conventions

- Commit/PR work from the **root of the repo** (no `git -C`); create branches with `git switch -c`.
- **NEVER** add `Co-Authored-By` trailers (or any "Generated with"/tool-attribution lines) to commit messages. Keep commit messages clean — subject + body only.
- Do **not** push or open PRs unless explicitly asked.
- Java 17+ and Gradle (`./gradlew`) build the project; see `AGENTS.md` for module targets and the `e2e/` fixtures.
