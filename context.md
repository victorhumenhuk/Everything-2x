# Decision Ledger — Everything-2x

Durable record of the significant decisions made in this repository and the reasoning behind them.

- **Confirmed** decisions are human-reviewed and binding. This section is maintained by the repository owner; the automated decision-ledger pass never edits it.
- **Inferred** decisions are hypotheses proposed automatically from the code, commit history, and any agent instructions (CLAUDE.md / AGENTS.md). They are **not binding** until the owner moves them into Confirmed.

## Confirmed

_None yet. Merge a proposal from Inferred to confirm it._

## Inferred (proposed — awaiting confirmation)

> Every item below is a hypothesis generated automatically on 2026-06-21. Where the rationale could not be recovered from the available evidence it is marked "rationale unknown — please supply".

### [hypothesis] Build as a Chrome Manifest V3 extension
- **Decision:** The product is a browser extension built on Chrome's Manifest V3 (`manifest_version: 3`), targeting Chrome 111+.
- **Rationale (hypothesis):** Manifest V3 is the only manifest version Chrome accepts for new Web Store submissions; `minimum_chrome_version: 111` is set explicitly because the extension relies on MAIN-world content scripts (the `"world": "MAIN"` declaration), which require Chrome 111 or newer (noted in CHANGELOG 2.0.1 "to ensure main-world content script support").
- **Evidence:** `manifest.json` lines 2, 6 (`manifest_version`, `minimum_chrome_version`); `CHANGELOG.md` 2.0.1 "Added `minimum_chrome_version: 111`"; `README.md` "Manifest V3 declaration"
- **First observed:** 20e4f2a (2026-04-06)

### [hypothesis] Two-script content architecture split across MAIN and isolated worlds
- **Decision:** Media handling is split into a MAIN-world script (`media-hook.js`) that monkeypatches `HTMLMediaElement.prototype.play` and `Element.prototype.attachShadow`, plus an isolated-world script (`shared.js` + `content.js`) that scans/applies rates and watches the DOM; the two communicate via `window.postMessage` with a tagged source.
- **Rationale (hypothesis):** Setting `playbackRate`/`defaultPlaybackRate` reliably (before media starts and against sites that reset it) requires patching the prototype in the page's own JS context, which only a MAIN-world script can reach; the isolated world is used for the privileged scanning/MutationObserver logic. The postMessage channel bridges the two worlds since they cannot share variables.
- **Evidence:** `manifest.json` content_scripts (MAIN world `media-hook.js` at document_start vs isolated `shared.js`/`content.js`); `media-hook.js` lines 111-152 (prototype patching), 154-166 (postMessage listener); `README.md` Architecture section; `shared.js` `PAGE_MESSAGE_SOURCE`/`PAGE_MESSAGE_TYPES`
- **First observed:** 20e4f2a (2026-04-06)

### [hypothesis] Minimal-permissions security posture (`storage` + `<all_urls>` host access only)
- **Decision:** Request only the `storage` permission plus `<all_urls>` host access; explicitly forbid `tabs`, `activeTab`, `scripting`, `webNavigation`, `cookies`, and `webRequest`. A CI step fails the build if any forbidden permission appears in `manifest.json`.
- **Rationale (hypothesis):** A prior `tabs` permission triggered a Chrome Web Store rejection (CHANGELOG 2.0.1 references "Chrome Web Store rejection reference Purple Potassium"); the unused `scripting`/`webNavigation` were dropped at the same time. The forbidden-permission CI gate enforces the minimal posture so a regression cannot ship.
- **Evidence:** `manifest.json` lines 14-19 (`permissions`, `host_permissions`); `.github/workflows/verify.yml` "Check forbidden permissions are absent"; `README.md` "No `tabs`, no `activeTab`, no `scripting`, no `webNavigation`"; `CHANGELOG.md` 2.0.1 Removed section
- **First observed:** 20e4f2a (2026-04-06); permission narrowing recorded in CHANGELOG 2.0.1

### [hypothesis] Settings stored in `chrome.storage.sync` with `local` fallback, default speed 2x enabled
- **Decision:** User settings (`enabled`, `speed`) are persisted in `chrome.storage.sync`; if sync fails the service worker falls back to `chrome.storage.local`. Defaults are `{ enabled: true, speed: 2 }`, with speed clamped to 0.25–4 in 0.05 steps.
- **Rationale (hypothesis):** Sync storage lets settings follow the user across devices (README "Settings sync across your devices via Chrome sync storage"); the local fallback guards against sync being unavailable/quota-limited. Default 2x matches the product name "Everything 2x".
- **Evidence:** `shared.js` lines 9-21 (`DEFAULT_SETTINGS`, `STORAGE_KEYS`, speed bounds), 43-48 (`clampSpeed`); `background.js` lines 46-57 (sync read with local fallback), 143-147 (onInstalled local fallback); `README.md` feature list
- **First observed:** 20e4f2a (2026-04-06)

### [hypothesis] No-data-collection / no-remote-code privacy stance
- **Decision:** The extension collects no data, contacts no external servers, ships no remote code, and uses no analytics/telemetry. This is documented in a dedicated PRIVACY.md and the README.
- **Rationale (hypothesis):** rationale unknown — please supply. (The stance is clearly stated and consistent with the minimal-permission design, but the underlying motivation is not recorded in the available evidence.)
- **Evidence:** `PRIVACY.md`; `README.md` Privacy section (lines 19-28); `manifest.json` description "No tracking; settings stay in sync storage"; commits d6ab3ab / ad9a1c7 (Privacy Policy add/update)
- **First observed:** d6ab3ab (2026-04-06)

### [hypothesis] No funding-solicitation links, single X-handle attribution, enforced by CI
- **Decision:** The project ships with no funding-solicitation links of any kind; attribution is limited to the author's name and X handle (@victorhumenhuk). A CI step greps every `.js`, `.html`, `.css`, `.json` and `.md` file against a blocklist of funding-platform names and phrasings, and fails the build if any reappears. Note: that grep covers this ledger too, so the blocked terms are deliberately not spelled out here — see `verify.yml` for the authoritative list.
- **Rationale (hypothesis):** rationale unknown — please supply. (CHANGELOG 2.0.2 records removal of "All external funding links" and the CI gate prevents reintroduction, but the reason for removing them is not stated.)
- **Evidence:** `.github/workflows/verify.yml` "Check no funding-link references remain"; `CHANGELOG.md` 2.0.2 Removed; `README.md` Author section; commit 04152ad "X attribution"
- **First observed:** 04152ad (2026-05-05)

### [hypothesis] CI verification on every push/PR to main
- **Decision:** A GitHub Actions `verify` workflow runs on push and pull_request to `main`, validating manifest JSON, forbidden permissions, version presence, absence of funding links, JS syntax (`node --check`), and required icon files.
- **Rationale (hypothesis):** rationale unknown — please supply. (The checks plainly guard the Web Store-relevant invariants, but no explicit reasoning is recorded.)
- **Evidence:** `.github/workflows/verify.yml`
- **First observed:** 3a15f71 / 04152ad (2026-05-05)

### [hypothesis] Tag-driven release pipeline with shell packaging script
- **Decision:** Releases are produced by pushing a `v*.*.*` git tag, which triggers a GitHub Actions `release` workflow that runs `scripts/package.sh` to build a zip and uploads it to a GitHub Release. The packaging script excludes git, docs, scripts, build dirs, and editor files from the shipped zip.
- **Rationale (hypothesis):** rationale unknown — please supply. (The tag-driven flow and exclusion list are documented in CONTRIBUTING.md and the workflow, but the motivation for this particular packaging approach is not stated.)
- **Evidence:** `.github/workflows/release.yml`; `scripts/package.sh`; `CONTRIBUTING.md` "Releasing a new version"
- **First observed:** 04152ad (2026-05-05)

### [hypothesis] Dependency-free, plain-JS implementation (no build step, no node_modules)
- **Decision:** The extension is written in plain browser JavaScript with no bundler, no framework, and no runtime dependencies; `node_modules/` and build tooling are only referenced defensively in `.gitignore` "in case ever added".
- **Rationale (hypothesis):** rationale unknown — please supply. (There is no package.json and the shared module uses a hand-rolled UMD wrapper, indicating a deliberate no-dependency choice, but the reasoning is not recorded.)
- **Evidence:** absence of `package.json`; `shared.js` lines 1-8 (UMD wrapper); `.gitignore` "Node (in case ever added)"; `background.js` line 1 `importScripts("shared.js")`
- **First observed:** 20e4f2a (2026-04-06)

### [hypothesis] Lightweight conventional commit style and rebase-first workflow
- **Decision:** Commits use a lightweight conventional prefix scheme (`Fix:`, `Feat:`, `Refactor:`, `Docs:`, `Chore:`) and the daily workflow rebases before work (`git pull --rebase`).
- **Rationale (hypothesis):** rationale unknown — please supply. (Documented as a personal workflow convention in CONTRIBUTING.md; no further reasoning given.)
- **Evidence:** `CONTRIBUTING.md` "Commit message style" and "Daily flow"; commit subjects (e.g. "Chore: add repository hygiene")
- **First observed:** 04152ad (2026-05-05)

---
*Decision-ledger automated pass. Operation: Bootstrap. Last reflection: commit `04152ad` (2026-06-21). Decisions above are AI-inferred hypotheses; nothing is binding until merged into Confirmed.*
