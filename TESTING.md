# RecipeJumper — Test Plan & Results

## 1. Automated unit tests (detector heuristics)

Source: [`test/detector.test.js`](test/detector.test.js) (Jest + jsdom) and the
equivalent in-browser runner [`test/browser-test.html`](test/browser-test.html).

Run with Node: `npm install && npm test`
Run without Node: `npm run serve` → open `http://localhost:8080/`

**Result: 11 / 11 passing** (verified in a real browser via the in-browser runner)

| # | Test | Result | Detail |
|---|------|--------|--------|
| 1 | WP Recipe Maker card | ✅ PASS | `type=plugin, label="Best Chocolate Chip Cookies", score=10` |
| 2 | schema.org microdata | ✅ PASS | `label="Fluffy Pancakes", score=10` |
| 3 | JSON-LD name parsing | ✅ PASS | `ld name="Lemon Drizzle Cake"` |
| 4 | JSON-LD inside `@graph` | ✅ PASS | `found Graph Curry` |
| 5 | Plain-blog headings fallback | ✅ PASS | `type=heading-ingredients, score=6` |
| 6 | Non-recipe page yields nothing | ✅ PASS | `0 candidates` |
| 7 | Recipe roundup → multiple | ✅ PASS | `Recipe One, Recipe Two` |
| 8 | Dedupe nested ingredients heading | ✅ PASS | `1 candidate "Tomato Soup"` |
| 9 | Hidden duplicate card skipped | ✅ PASS | `kept "Visible Recipe"` |
| 10 | `hostMatches` patterns | ✅ PASS | exact / `*.sub` / wildcard / non-match |
| 11 | `isExcluded` | ✅ PASS | matches across patterns |

## 2. End-to-end UI tests (real content script in a browser)

Driven against [`test/e2e-demo.html`](test/e2e-demo.html) (single recipe) and
[`test/e2e-multi.html`](test/e2e-multi.html) (two recipes), loading the actual
`content.js` / `detector.js` / `settings.js` / `content.css`.

| Behavior | Result | Evidence |
|----------|--------|----------|
| Floating button renders in Shadow DOM | ✅ | `#recipejumper-root` + `.rj-fab` present, no style bleed |
| Single recipe → exactly 1 candidate | ✅ | `[{plugin, "Grandma's Famous Cinnamon Rolls", score 13}]` |
| Own UI not mis-detected as a recipe | ✅ | fixed `[id*='recipe']` matching `recipejumper-root`; observer ignores own DOM |
| Click button → scroll to recipe | ✅ | `scrollTo top=5171` == expected; lands with 12px offset |
| Post-jump highlight applied | ✅ | `.rj-highlight` added to recipe element |
| Multi-recipe → button shows pick state | ✅ | label "2 recipes found — choose one" |
| Menu opens with named items + focus mgmt | ✅ | 2 items, `aria-expanded=true`, focus on first item |
| Keyboard nav (↓ wraps, Esc closes → focus FAB) | ✅ | all transitions correct |
| Menu item → jumps to that recipe | ✅ | 2nd item scrolls to `top=1632` == expected |
| In-page shortcut `Alt+Shift+R` | ✅ | triggers jump to correct position |
| Wrong combo ignored | ✅ | plain `R` does nothing |
| Shortcut ignored while typing in a field | ✅ | keydown in `<input>` does nothing |
| No console errors throughout | ✅ | console clean across all interactions |

## 3. Manual cross-browser checklist

Use a few live sites (e.g. AllRecipes, Food Network, BBC Good Food, Serious
Eats, and a couple of personal food blogs) per browser.

- [ ] Button appears on recipe pages, hidden on non-recipe pages
- [ ] Jump is smooth and lands on the ingredients/recipe section
- [ ] Highlight flash is visible and brief
- [ ] Roundup pages show the multi-recipe menu
- [ ] Right-click → *Jump to Recipe* works
- [ ] Right-click → *Set this section as the recipe* fixes a missed page and persists on reload
- [ ] In-page and native shortcuts both trigger the jump
- [ ] Theme (light/dark/auto) and button position apply live
- [ ] Per-site exclusion hides the button on that host
- [ ] Settings persist across browser restarts
- [ ] No console errors

| Browser | Engine | Manifest | Status |
|---------|--------|----------|--------|
| Chrome / Edge / Brave | Chromium | MV3 (`src/`) | Ready to load unpacked |
| Firefox (current) | Gecko | MV3 (`src/`) | Ready (temporary add-on / signed) |
| Firefox (legacy) | Gecko | MV2 (`manifest.v2.json`) | Build via `npm run build` |
| Safari | WebKit | MV3 (converted) | Requires macOS + Xcode conversion |

> Automated unit + E2E tests were executed in a Chromium-based runner on this
> machine. Live multi-site and Firefox/Safari runs require those browsers; the
> checklist above is the script to follow there.

## 4. Performance notes

- Detection is debounced (600 ms) and the MutationObserver ignores the
  extension's own DOM, so there's no rescan loop.
- The highlight uses a small GPU-friendly glow (no full-page box-shadow spreads),
  keeping repaint cheap on content-heavy pages.
- The FAB and menu live in a Shadow DOM — zero CSS conflicts in either direction.
