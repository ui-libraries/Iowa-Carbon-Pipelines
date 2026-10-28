# Iowa CO2 Pipelines map: accessibility and upgrade changes

**Files changed:** `index.html`, `src/leaflet-sidebar.css`
**Target standard:** WCAG 2.1 Level AA (the standard the ADA Title II web rule points to)

> **Important:** This is a thorough remediation, but it is not a certification. I could not test in a real browser or with a screen reader, so please run the testing checklist at the bottom before declaring the site compliant. A smoke test of the page script against fixture data (jsdom) passed with no errors.

---

## 1. Library and basemap changes

| Change | Before | After |
|---|---|---|
| Leaflet | 1.7.1 | **1.9.4** (current stable; 2.0 is still alpha), with SRI hashes computed from the published npm package |
| Basemap | CARTO Positron (`basemaps.cartocdn.com/light_all`) | **OpenStreetMap standard tiles** (`tile.openstreetmap.org`) |
| Attribution | OSM + CARTO | OSM only (required by OSM's license) |
| Zoom limits | min 7, max 11 | unchanged |

Notes on the basemap:
- OSM tiles are free and need no API key. For a university site with modest traffic, OSM's [tile usage policy](https://operations.osmfoundation.org/policies/tiles/) is fine. If traffic grows a lot, move to a hosted free-tier provider or self-host tiles.
- OSM is visually busier than Positron. The county outlines (black) and fills still read well, but check the look. I darkened the pipeline colors (below) partly for this reason.
- Leaflet 1.9 also includes accessibility improvements over 1.7 (for example, labeled popup close and zoom buttons and more robust keyboard handling), and it is the line that receives security fixes.

**Compatibility risk to test:** the two plugins not included in the upload (`leaflet-sidebar.js` from jsdelivr and your local `src/leaflet.activearea.js`) were written for Leaflet 1.x. I read both. The sidebar plugin only uses public APIs. The active-area plugin overrides a few private methods (`GridLayer._resetView`, `_update`), which still exist in 1.9.4. Please confirm that **Search** still recenters the county in the visible (non-sidebar) part of the map.

---

## 2. Accessibility changes

### Structure and landmarks (WCAG 1.3.1, 2.4.1, 2.4.2, 3.1.1)
- Added a **"Skip to map"** link (visible on keyboard focus).
- Added `<main>` and a visually hidden `<h1>` (the original `<h1>` was inside the sidebar and disappeared whenever the sidebar was collapsed).
- Sidebar is now an `<aside>` with a label.
- Fixed heading hierarchy: page `h1` → sidebar `h2` → sections `h3`. Previously `<h3>` and `<h4>` were used for body paragraphs and the numbered instructions, and `<h2>` sat inside `<p>` (invalid HTML).
- Instructions are now a real `<ol>`; paragraphs are `<p>`; removed a stray `</b>` and the `<br>` spacing hacks.
- Added a `<noscript>` message.

### Sidebar and keyboard operation (WCAG 2.1.1, 2.4.3, 4.1.2)
- **Collapsed sidebar content was still keyboard-focusable.** The plugin only clips it with `overflow:hidden`. It is now hidden with `visibility:hidden` (delayed so the collapse animation still plays). The same fix removes a custom `.sidebar-pane { display:flex }` override that kept inactive panes rendered.
- The sidebar toggle is now `role="button"` with an accessible name, `aria-expanded`, and `aria-controls`; Space works as well as Enter. It stays in sync with the sidebar state.
- The **close control was a non-focusable `<span>`**. It is now a `<button>` with a label. Focus moves to the toggle when the sidebar closes.
- Removed the invalid `role="tablist"` / `role="tab"` (there is only one toggle, not a tab set).
- "About This Project" is now a proper disclosure: a `<button>` inside the heading with `aria-expanded` and `aria-controls`. It uses the `hidden` attribute instead of inline styles.
- Decorative Font Awesome icons are `aria-hidden`.

### Forms (WCAG 1.3.1, 3.3.2, 2.5.3, 4.1.2)
- Radio buttons are in a `<fieldset>` with a `<legend>`.
- The county `<select>` now has a real `<label>`. Previously it had none, and the JavaScript injected a second `<select>` *inside* the first (invalid). It is now built once with plain `<option>`s (the old code rebuilt it for every county polygon).
- Controls are in one `<form>`: Enter in the menu now runs the search. The buttons have `type`s. Their accessible names contain the visible text ("Search for the selected county", "Zoom Out to all of Iowa").
- Fixed the label that pointed at the wrong radio (`for="litigation"` on the utility option, currently commented out).
- Search results are in a `<section>` with `aria-live="polite"`, so screen readers hear the results. A load-failure message is shown there if the data files fail to load.

### Map interaction (WCAG 2.1.1, 2.1.2, 2.4.7, 1.4.13)
- County shapes on the **ordinance, litigation, and news layers are now keyboard operable**: they are focusable, have a text label, open on Enter or Space, show their tooltip and a thicker outline on focus, and (for litigation) move focus into the popup and back on close.
- **Zoom buttons restored** (`topright`). They had been removed, leaving mouse and touch users without a visible zoom control. They sit top-right because the sidebar covers the top-left.
- Layer control is labeled "Map layers" (`role="group"`).
- Instructions no longer rely only on location ("upper right corner"); they also name the control (WCAG 1.3.3). Added a keyboard-usage instruction.
- The sidebar county search is the **equivalent, text-based way** to get all ordinance, litigation, and news content, so keyboard and screen reader users are not forced through the map.
- Bug fix: the ordinance click handler used an undefined `event` variable. It now uses the handler's own parameter.
- New-tab links: static links and links inside the data (e.g. `Ordinance_Hyperlinks`, `news1…13`) get `rel="noopener noreferrer"` and a screen-reader-only "(opens in a new tab)". Tooltips now say links open in new tabs. `window.open` uses `noopener`.
- Fixed `<a href="brupe@uiowa.edu">` (it was a broken relative link) to `mailto:`.

### Color and contrast (WCAG 1.4.1, 1.4.3, 1.4.11)
| Item | Before | After |
|---|---|---|
| Active sidebar tab | white on tan, **1.4:1** | black on tan, **14.7:1** |
| Pipeline lines | red / orange / green, differing **only by color**; orange was **1.7:1** on the map | `#b8330a` solid, `#5e3c99` dashed, `#006d2c` dotted. All **5.2:1 or better** on the OSM background, and distinguishable without color |
| Pipeline names | none (tooltips were commented out) | hover tooltip names each line; legend states "(solid / dashed / dotted line)" |
| Legend | white text on translucent gray (about 4.6:1, varied with the map behind it) | dark text on a near-opaque white panel (17:1) with a border; swatches are outlined |
| Keyboard focus ring | browser default, often clipped or invisible | 3px `#005fcc` ring (6.0:1 on white, 5.0:1 on the tan sidebar) on all controls |
| Body font | Poppins 300 (thin), 12–13px | Poppins 400, sized in `rem` (14–15px) so it scales with browser text-size settings |

The **pipeline color change is the most visible design change**. If your style guide needs different hues, keep the dash patterns and check any new color for 3:1 against the map.

### Images (WCAG 1.1.1)
- Logos had `alt="Item 1"` … `"Item 4"`. They are now: Hubbell Environmental Law Initiative; University of Iowa College of Law; University of Iowa Law Library; Digital Scholarship & Publishing Studio, University of Iowa Libraries. **Please verify the Law Library wording**, which I inferred from the filename.
- Legend swatches are `aria-hidden` because the text beside them carries the meaning.

### Motion (WCAG 2.3.3)
- `prefers-reduced-motion` is honored: Leaflet zoom/fade animations and `flyToBounds` are turned off, and CSS transitions are removed (in both `index.html` and `leaflet-sidebar.css`).

---

## 3. Code cleanup that came with the fixes
- Four near-identical 5-to-13-branch `if/else` chains for litigation and news were replaced by loops and one shared `renderCountyResult()`. Output markup is now valid (`<ul><li>…</li></ul>`; before it was `<li>` with stray `<br>`s and invalid nesting).
- `info.innerHTML = chapterContent` (an array coerced to a comma-joined string) now uses `.join('')`.
- Removed unused variables (`modal`, `legendContainer`, etc.).
- Guarded Search / Zoom Out against being clicked before the data loads.

**Behavior change to be aware of:** the old chains showed only the *first unbroken run* of fields. For example, if a county had `news1` and `news3` but no `news2`, `news3` was silently dropped. The loops show every non-empty field.

---

## 4. Not changed / still needs your attention
- **Pipeline routes have no text equivalent.** The lines can only be seen on the map. Consider adding a fourth search option for "Planned Pipeline Counties" (the data is already loaded as `pipeline === "yes"`) or a short written description of each route.
- **Link text inside the data files** (`Ordinance_Hyperlinks`, `news1…13`) comes from your GeoJSON. If any link text is a bare URL or "click here", edit it in the data so each link makes sense out of context.
- **Tab stops:** each county with an ordinance, litigation, or news item is now a tab stop on the map when that layer is shown. That can be many Tab presses, which is why the sidebar search is documented as the quickest keyboard route.
- Google Analytics tags, Font Awesome, d3 v4, and jQuery were left as-is. jQuery appears unused in this file. d3 v4 is old but works. The Font Awesome CDN link has no SRI hash.
- Typo left in the content: "Research for this project was **conducting** by…". I did not edit your copy.
- The sidebar's fixed pixel widths and the logo strip's negative margins were left alone. They are worth checking at 200% zoom and at 320px width (WCAG 1.4.10 Reflow).
- Files I did not receive (`src/leaflet.activearea.js`, `data/*.geojson`, `lockups/*.svg`) were not modified.

---

## 5. Suggested testing checklist
1. **Keyboard only:** Tab from the top. Check skip link, sidebar toggle, the About toggle, radios (arrow keys), county menu, Search, Zoom Out, map zoom buttons, layer control, and a focused county shape (Enter opens links or popup).
2. **Screen reader** (NVDA + Firefox/Chrome, VoiceOver + Safari): confirm the toggle announces "Open/Close sidebar, collapsed/expanded", search results are announced, and collapsed sidebar content is not reachable.
3. **Zoom to 200% and 400%** and check for clipped or overlapping content.
4. **Reduced motion:** enable "reduce motion" in your OS and confirm no fly/zoom animation.
5. **Automated scan:** run axe DevTools or WAVE on both the collapsed and expanded sidebar states, and with each radio layer selected.
6. **Regression:** confirm Search centers the county in the visible map area (active-area plugin), and the zoom buttons don't overlap the layer control.
