# Bravo — an internal fork of [Maestro](https://github.com/mobile-dev-inc/Maestro)

**Bravo** extends [Maestro](https://github.com/mobile-dev-inc/Maestro) for driving canvas-based UIs (e.g. Lightning/WebGL TV apps). It tracks upstream Maestro and adds Bravo-specific capabilities.

---

## Fork commit registry

Bravo is a **minimal, linear stack of feature commits on top of `upstream/main`** (the original Maestro). The table below registers every fork commit so the delta to upstream is always auditable. It is keyed on **commit subject + purpose, not SHA** — rebasing on upstream rewrites the SHAs, but the subjects are stable. Keep commits minimal: prefer amending an existing commit over adding a new one. See [`CLAUDE.md`](CLAUDE.md) for the full workflow.

| # | Commit subject | Purpose | Upstream |
|---|----------------|---------|----------|
| 1 | `docs: create Bravo as fork of Maestro` | README rebrand, `CLAUDE.md`, the `update-from-upstream` skill, this registry, and the `major.minor.patch.build` versioning scheme (`CLI_VERSION` tracks Maestro; the fork-owned `BRAVO_BUILD` adds a 4th segment, auto-incremented at release time by `publish-cli`, so fork-only fixes ship without drifting from upstream). | — (fork-only) |
| 2 | `fix(web): prefer data-testid for element selection` | Web driver selects elements by `data-testid` first for stabler Lightning/WebGL selection. | not yet upstreamed |
| 3 | `feat(tvos): add apple tv support` | Apple TV (tvOS) driver, device handling, RN Expo tvOS demo app, and tvOS e2e flows. | [PR #3021](https://github.com/mobile-dev-inc/Maestro/pull/3021) |
| 4 | `feat(web): expand web driver keyboard support` | Maps `REMOTE_DPAD` keycodes (arrows + center) to Selenium arrow keys for Arrow-based navigation on Lightning web apps. | not yet upstreamed |
| 5 | `feat(dist): distribute Bravo via houwert.dev` | Releases land on `DouweBos/Bravo`; installer (`curl -fsSL https://houwert.dev/bravo/get \| bash`) and CLI update-check/changelog point at the houwert.dev reverse proxy instead of upstream Maestro. | fork-only |
| 6 | `feat(web): detect web flows from a URL-shaped appId` | `FileUtils.isWebFlow()` also treats an `http(s)://` `appId` as a web target, so a flow can drive the web driver via `appId` alone (no separate `url:` field). | not yet upstreamed |
| 7 | `feat(web): expose focus state to the focused selector` | Emits `focused` into the typed `TreeNode.focused` from `document.activeElement` (real web focus) or a `data-focused` flag, so `focused: true` selectors resolve on the web driver — including canvas UIs (Lightning), where the canvas itself is always the active element. | not yet upstreamed |
| 8 | `fix(web): honor -p web when selecting the web device` | An explicit `--platform web` forces web-device inclusion, so flows whose `appId` is a runtime variable (resolved via `-e APP_ID=…`) pick the web driver instead of failing with "0 devices connected". | not yet upstreamed |

### Remotes / staying in sync

| Remote | URL | Role |
|--------|-----|------|
| `origin` | `https://github.com/DouweBos/Bravo.git` | The fork; `main` is canonical. |
| `upstream` | `https://github.com/mobile-dev-inc/Maestro.git` | Original Maestro; we rebase onto `upstream/main`. |

To pull in the latest Maestro, rebase the fork on upstream — use the [`update-from-upstream`](.claude/skills/update-from-upstream/SKILL.md) skill (Claude Code), or manually: `git fetch upstream && git rebase upstream/main && git push --force-with-lease origin main`.

---

> [!TIP]
> Great things happen when testers connect — [Join the Maestro Community](https://maestrodev.typeform.com/to/FelIEe8A)

<p align="center">
  <a href="https://www.maestro.dev">
    <img width="1200" alt="Maestro logo" src="https://github.com/mobile-dev-inc/Maestro/blob/main/assets/banne_logo.png" />
  </a>
</p>

<p align="center">
  <strong>Maestro</strong> is an open-source framework that makes UI and end-to-end testing for Android, iOS, and web apps simple and fast.<br/>
  Write your first test in under five minutes using YAML flows and run them on any emulator, simulator, or browser.
</p>

<img src="https://user-images.githubusercontent.com/847683/187275009-ddbdf963-ce1d-4e07-ac08-b10f145e8894.gif" />

---

## Table of Contents

- [Why Maestro?](#why-maestro)
- [Getting Started](#getting-started)
- [Resources & Community](#resources--community)
- [Contributing](#contributing)
- [Maestro Studio – Test IDE](#maestro-studio--test-ide)
- [Maestro Cloud – Parallel Execution & Scalability](#maestro-cloud--parallel-execution--scalability)

---

## Why Maestro?

Maestro is built on learnings from its predecessors (Appium, Espresso, UIAutomator, XCTest, Selenium, Playwright) and allows you to easily define and test your Flows.

By combining a human-readable YAML syntax with an interpreted execution engine, it lets you write, run, and scale cross-platform end-to-end tests for mobile and web with ease.

- **Cross-platform coverage** – test Android, iOS, and web apps (React Native, Flutter, hybrid) on emulators, simulators, or real devices.
- **Human-readable YAML flows** – express interactions as commands like `launchApp`, `tapOn`, and `assertVisible`.
- **Resilience & smart waiting** – built-in flakiness tolerance and automatic waiting handle dynamic UIs without manual `sleep()` calls.
- **Fast iteration & simple install** – flows are interpreted (no compilation) and installation is a single script.

**Simple Example:**

```
# flow_contacts_android.yaml

appId: com.android.contacts
---
- launchApp
- tapOn: "Create new contact"
- tapOn: "First Name"
- inputText: "John"
- tapOn: "Last Name"
- inputText: "Snow"
- tapOn: "Save"
```

---

## Getting Started

Maestro requires Java 17 or higher to be installed on your system. You can verify your Java version by running:

```
java -version
```

Installing the CLI:

Run the following command to install **Bravo** on macOS, Linux or Windows (WSL):

```
curl -fsSL "https://houwert.dev/bravo/get" | bash
```

This installs the Bravo CLI (a drop-in `maestro` command) from the Bravo distribution — see [Fork commit registry](#fork-commit-registry) for how it's served. To install a specific version, set `MAESTRO_VERSION` (e.g. `MAESTRO_VERSION=2.6.1`) before running.

> Upstream Maestro installs via `curl -fsSL "https://get.maestro.mobile.dev" | bash` — use the Bravo URL above instead to get the Bravo multi-platform build.

The links below will guide you through the next steps.

- [Installing Maestro](https://docs.maestro.dev/getting-started/installing-maestro) (includes regular Windows installation)
- [Build and install your app](https://docs.maestro.dev/getting-started/build-and-install-your-app)
- [Run a sample flow](https://docs.maestro.dev/getting-started/run-a-sample-flow)
- [Writing your first flow](https://docs.maestro.dev/getting-started/writing-your-first-flow)

---

## Resources & Community

- 💬 [Join the Slack Community](https://maestrodev.typeform.com/to/FelIEe8A)
- 📘 [Documentation](https://docs.maestro.dev)
- 📰 [Blog](https://maestro.dev/blog?utm_source=github-readme)
- 🐦 [Follow us on X](https://twitter.com/maestro__dev)

---

## Contributing

Maestro is open-source under the Apache 2.0 license — contributions are welcome!

- Check [good first issues](https://github.com/mobile-dev-inc/maestro/issues?q=is%3Aopen+is%3Aissue+label%3A%22good+first+issue%22)
- Read the [Contribution Guide](https://github.com/mobile-dev-inc/Maestro/blob/main/CONTRIBUTING.md)
- Fork, create a branch, and open a Pull Request.

If you find Maestro useful, ⭐ star the repository to support the project.

---

## Maestro Studio – Test IDE

**Maestro Studio Desktop** is a lightweight IDE that lets you design and execute tests visually — no terminal needed.
It is also free, even though Studio is not an open-source project. So you won't find the Maestro Studio code here.

- **Simple setup** – just download the native app for macOS, Windows, or Linux.
- **Visual flow builder & inspector** – record interactions, inspect elements, and build flows visually.
- **AI assistance** – use MaestroGPT to generate commands and answer questions while authoring tests.

[Download Maestro Studio](https://maestro.dev/?utm_source=github-readme#maestro-studio)

---

## Maestro Cloud – Parallel Execution & Scalability

When your test suite grows, run hundreds of tests in parallel on dedicated infrastructure, cutting execution times by up to 90%. Includes built-in notifications, deterministic environments, and complete debugging tools.

Pricing for Maestro Cloud is completely transparent and can be found on the [pricing page](https://maestro.dev/pricing?utm_source=github-readme).

👉 [Start your free 7-day trial](https://maestro.dev/cloud?utm_source=github-readme)

```
  Built with ❤️ by Maestro.dev
```
