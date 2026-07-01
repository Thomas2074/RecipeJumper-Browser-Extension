# RecipeJumper

**Skip the life story. Jump straight to the recipe.**

RecipeJumper is a lightweight, privacy-friendly cross-browser extension that
detects the recipe on any page and lets you jump to it instantly — with one
click, a keyboard shortcut, or the right-click menu. No bundler, no tracking,
no accounts.

- **One-click jump** via a floating button that only appears when a recipe is found — always goes straight to the best-matched recipe, no menus
- **Keyboard shortcut** (`Alt+Shift+R` by default, fully customizable)
- **Right-click** → *Jump to Recipe*, or *Set this section as the recipe* for stubborn pages
- **Light / dark / auto theme**, configurable button position
- **Per-site exclusions** with wildcard support
- **Accessible** — keyboard navigable, focus-managed, screen-reader friendly, honors reduced-motion
- **No network calls, no analytics** — everything runs locally

---

## How it works

Detection runs entirely in the page using layered heuristics, strongest first:

1. **`schema.org/Recipe` microdata** containers (`itemtype`, `itemprop="recipeIngredient"`)
2. **JSON-LD** `@type: Recipe` (including `@graph` arrays) — used as a strong confidence signal
3. **Known recipe-plugin cards** — WP Recipe Maker, Tasty Recipes, Mediavine Create, etc.
4. **Ingredient list ancestors** (`itemprop="recipeIngredient"`)
5. **"Ingredients" / "Instructions" headings** — the fallback for plain blogs

Each candidate gets a confidence score; overlapping signals are de-duplicated, our
own UI is excluded, and hidden/print-only duplicates are filtered out. The full
logic lives in [`src/lib/detector.js`](src/lib/detector.js) and is pure/DOM-only so
it can be unit-tested.

---

## Install (development / unpacked)

The extension needs **no build step** — load the `src/` folder directly.

### Google Chrome / Edge / Brave (Manifest V3)
1. Go to `chrome://extensions`
2. Enable **Developer mode** (top right)
3. Click **Load unpacked** and select the **`src/`** folder
4. Pin the RecipeJumper icon and visit any recipe page

### Mozilla Firefox (Manifest V3)
1. Go to `about:debugging#/runtime/this-firefox`
2. Click **Load Temporary Add-on…**
3. Select **`src/manifest.json`**
   - For a permanent install, use a signed build from [AMO](https://addons.mozilla.org/developers/) or `web-ext sign`.
   - Older Firefox? Use the legacy MV2 build (see below).

### Apple Safari (macOS only)
Safari requires native packaging with Xcode, which **cannot be done on Windows**.
On a Mac:
```bash
xcrun safari-web-extension-converter ./src
```
This produces an Xcode project; build & run it, then enable the extension in
**Safari → Settings → Extensions**. The MV3 source is Safari-compatible
(`browser`/`chrome` namespace is normalized in [`src/lib/settings.js`](src/lib/settings.js)).

---

## Usage

| Action | How |
| --- | --- |
| Jump to recipe | Click the floating down-arrow button, or press `Alt+Shift+R` |
| Jump from anywhere | Right-click → **Jump to Recipe** |
| Fix a missed recipe | Right-click the recipe heading → **Set this section as the recipe** (remembered per page) |
| Settings | Click the toolbar icon → **All settings** |

### Customizing the shortcut
- **In-page shortcut** (works in every browser): Settings → *Keyboard shortcut* → click and press your combo.
- **Native browser shortcut** (works even when the page isn't focused): `chrome://extensions/shortcuts` (Chrome) or *about:addons → Manage Extension Shortcuts* (Firefox).

> The default is `Alt+Shift+R` rather than the spec's `Ctrl+Shift+R`, because
> `Ctrl+Shift+R` is the browser's hard-reload shortcut and can't be reliably
> overridden.

---

## Permissions & privacy

| Permission | Why |
| --- | --- |
| `storage` | Save your preferences (synced across devices when available) |
| `activeTab` + `scripting` | Inject/trigger the jump on the page you're viewing |
| `contextMenus` | The right-click menu items |
| content script on `<all_urls>` | Needed to auto-detect recipes and show the button on any recipe page. It only reads the page's DOM locally. |

RecipeJumper makes **no network requests** and collects **no data**.

---

## Development

No toolchain is required to run the extension. The optional dev dependencies are
only for tests, linting, and signed builds.

```bash
npm install          # optional: jest + web-ext
npm test             # run detector unit tests (Jest + jsdom)
npm run lint         # web-ext lint of src/
npm run build        # package store-ready zips into dist/ (PowerShell, no Node needed)
npm run serve        # static preview server on http://localhost:8080 (no Node needed)
```

### Testing without Node
- **`test/browser-test.html`** runs the detector test suite in any browser — just open it (or run `npm run serve` and visit `http://localhost:8080/`).
- **`test/e2e-demo.html`** and **`test/e2e-multi.html`** are realistic recipe pages that load the real content scripts, so you can see the button, jump, and multi-recipe menu in action without installing anything.

See [TESTING.md](TESTING.md) for the test matrix and results.

---

## Project structure

```
.
├── src/                      # the extension (load this unpacked)
│   ├── manifest.json         # Manifest V3 (Chrome / Firefox / Safari)
│   ├── lib/
│   │   ├── settings.js       # shared defaults + storage helpers + host matching
│   │   └── detector.js       # pure recipe-detection heuristics (unit-tested)
│   ├── content/              # content script (FAB in Shadow DOM) + highlight CSS
│   ├── background/           # service worker: commands, context menu, injection
│   ├── popup/                # toolbar popup (status + quick settings)
│   ├── options/              # full settings page
│   └── icons/                # generated PNG icons
├── manifest.v2.json          # legacy Manifest V2 (old Firefox / Safari)
├── test/                     # Jest tests + in-browser runner + demo pages
├── scripts/                  # build.ps1 (packaging), serve.ps1 (preview)
└── package.json
```

---

## Cross-browser notes

- **Manifest V3 is the single source of truth.** Modern Chrome, Firefox, and
  Safari all support it. `browser_specific_settings.gecko` is included for
  Firefox; Chromium ignores it.
- **`manifest.v2.json`** is provided for legacy Firefox/Safari. `npm run build`
  emits a `*-firefox-legacy-*.zip` with it swapped in. (In MV2 the on-demand
  injection fallback is unavailable, but the declared content script already
  covers every page, so it rarely matters.)
- The `browser` vs `chrome` namespace difference is normalized once in
  `settings.js`; storage falls back from `sync` to `local` automatically.

---

## Deployment

1. `npm run build` → `dist/recipejumper-chrome-<v>.zip`, `…-firefox-<v>.zip`, `…-firefox-legacy-<v>.zip`
2. **Chrome Web Store** → [Developer Dashboard](https://chrome.google.com/webstore/developer/dashboard) → upload the Chrome zip
3. **Firefox Add-ons (AMO)** → [Developer Hub](https://addons.mozilla.org/developers/) → upload the Firefox zip (or `web-ext sign`)
4. **Safari** → convert with `safari-web-extension-converter` on macOS, then submit via [App Store Connect](https://appstoreconnect.apple.com/)

---

## Roadmap

- [ ] Localized detection keywords (multi-language "Ingredients"/"Instructions")
- [ ] Optional NLP assist (e.g. [compromise](https://github.com/spencermountain/compromise)) for fuzzy blogs
- [ ] "Save recipe" / clip-to-clipboard
- [ ] Cloud preference sync via account (currently uses browser sync storage)

## License

[MIT](LICENSE) © 2026 deltaclint
