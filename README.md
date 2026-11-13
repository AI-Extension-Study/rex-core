# rex-core

The core of the rex framework for browser extensions, which every other rex module builds on. In the research browser extension that bundles it, rex-core runs the identifier screen, fetches and stores the served configuration, hands every collected record from the crawlers to the upload module, and keeps a small key-value store the crawlers use to remember what they sent.

**Where it stands, 2026-11-13.** Apart from this README, this fork's `main` is bric-digital's `main`, `e7cd111` (2026-10-28). The extension does not install this fork: it installs bric-digital's `e7cd111`, the same code.

## What it does in the extension

- **Identifier screen.** On save, the page fetches the stored `configuration_url` with `<IDENTIFIER>` replaced by the entered ID, URL-encoded. A 2xx JSON answer replaces the whole stored configuration, and only then is the ID stored, as `rexIdentifier` in `chrome.storage.local`. Any other answer shows an error and stores nothing. The page then shows the first screen in the configuration's `ui` list whose requirements pass; `has_identifier` passes once `rexIdentifier` is set.
- **Configuration refresh.** The `refreshConfiguration` message fetches `configuration_url` again, with the stored ID, not URL-encoded, in place of `<IDENTIFIER>`. A 2xx JSON answer replaces the stored configuration and every module is told to read it again; any other answer leaves it as it was. The configuration bundled with the extension is stored only when storage holds none (`loadInitialConfiguration`), so a new bundled one never reaches an existing install.
- **Service worker setup.** `setup()`, which the extension calls at every service-worker start, listens for `onInstalled`: on install and on every update (it does not check why the event fired) it records `rexInstallTime` if it is not set and opens the extension window. It also opens the window on a click of the toolbar button, loads the page scripts into the top frame of each http(s) page, and opens the IndexedDB database `rex_db` with one object store, `values`. Crawlers keep their upload keys there through the `storeValue` and `fetchValue` messages.
- **`dispatchEvent(event)`** hands an event to every module registered with this copy of rex-core. Crawlers call it for each record and each completion event, and the upload module receives them through its `logEvent`.
- **The extension window** is `index.html` in a 640 by 480 panel. Before opening it, rex-core closes every window that has a tab showing that page, with all of that window's other tabs.

## Pinning this fork

Pin it only to change it. The extension and all 13 other `@bric` packages it installs, the crawlers included, name `github:bric-digital/rex-core` as a dependency. A pin of this fork at the extension's root does not change what those modules ask for, so npm can install bric-digital's copy under each of them as well. Each copy keeps its own list of modules (`registeredExtensionModules` in `src/service-worker.mts`), so a record dispatched through one copy never reaches a module registered with another, such as the uploader, and nothing reports an error. A pin therefore needs an `overrides` entry in the extension's `package.json`, not yet tried with git dependencies, and a check that `npm ls @bric/rex-core` lists one copy.

## What is in this repo

| path | what it holds |
|---|---|
| `src/service-worker.mts` | setup, the messages it answers, configuration, storage, `dispatchEvent` |
| `src/extension.mts` | the extension pages: choosing a screen, the identifier screen |
| `src/browser.mts`, `src/common.mts` | the base for page scripts; shared types, SHA-256 and SHA-512, configuration scope |
| `tests/` | a test extension and Playwright specs |

`npm run build`, `npm run lint` and `npm test`, which starts Chromium and a local server on port 3000. `.circleci/config.yml` is bric-digital's CircleCI setup; no CI runs on this fork.

## Keeping this current

Update this README when this fork gains a change of its own, or when the extension starts installing it.
