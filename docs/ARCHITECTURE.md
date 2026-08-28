# Shina Fox architecture and compatibility

Shina Fox is a CSS-only browser-chrome theme. There is no build step, JavaScript
runtime, package manager, or generated output in this repository. Firefox/Floorp
parses the checked-in stylesheet against private browser UI markup, so that markup
is the project's effective API.

The currently audited baseline is Floorp 12.16.4 on Gecko 153 with Sidebery 5.6.1.
Private UI selectors are not a stable extension API; the baseline describes what
the theme is being migrated toward, not a compatibility guarantee for every
Firefox or Floorp release.

## Repository runtime

| Path | Runtime role |
| --- | --- |
| `floorp.css` | The complete browser UI theme. This is the only executable source file. |

The monolithic stylesheet is organized in this source order:

1. Global fonts, colors, and shared variables.
2. Navigation bar, titlebar controls, tabs, and context menus.
3. Extension toolbar controls.
4. Address bar and address-bar results.
5. Bookmarks toolbar.
6. Firefox's extension sidebar and Sidebery-specific behavior.
7. Floorp's panel sidebar.
8. Browser content frame and find bar.

Keep rules close to the matching section until a deliberate module-loading design
is introduced. A file split alone does not provide isolation: stylesheet order,
selector specificity, and `!important` declarations are part of the current
theme's behavior.

## Two CSS loading paths

### Firefox startup loading (the supported deployment)

Firefox looks for the exact file names below in the profile root:

```text
<profile>/
└── chrome/
    ├── userChrome.css   # Browser interface
    ├── userContent.css  # Internal/content documents, if used
    ├── dark.png
    └── light.png
```

`userChrome.css` is read at browser startup when
`toolkit.legacyUserProfileCustomizations.stylesheets` is `true`. Floorp currently
enables that preference in its packaged defaults, but portable instructions and
plain Firefox installations must still verify it in `about:config`. Root
`userChrome.css` changes require a full browser restart; Floorp's live reload does
not rebuild this file.

The file is named `userChrome.css`, not `chrome.css`. Likewise, `userChrome.js` is
not a native Firefox or Floorp feature. JavaScript loaders require a separate,
privileged AutoConfig/bootstrap installation and are intentionally outside this
project. Floorp's CSS loader does not execute JavaScript.

### Floorp live CSS loading (optional development path)

Floorp also scans direct `.css` children of `<profile>/chrome/CSS/` (capital
`CSS`), or the directory selected by `UserCSSLoader.FOLDER`. Its menu action named
“Edit userChrome.css” targets `chrome/CSS/userChrome.css`; despite the familiar
name, that is not the root startup stylesheet above.

Floorp classifies live-loaded files by name:

| File name | Registered sheet origin |
| --- | --- |
| Plain `.css` | User |
| `xul-*` or `*.as.css` | Agent |
| `*.author.css` | Author |
| `*.uc.css` | Excluded from this loader |

The app-menu rebuild action and `Alt+R` reload this directory only. These sheets
are registered more globally than root `userChrome.css`, so a broad rule such as
the theme's universal `*` font selector can leak into unintended UI documents.
Any live-loader experiment must be explicitly scoped, for example:

```css
@charset "UTF-8";

@-moz-document url(chrome://browser/content/browser.xhtml) {
  /* experiment */
}
```

If present, `@charset` must be the first statement. The exact `url(...)` scope
also deliberately excludes child documents whose URL has a query string, such as
a Floorp web-panel browser window.

Do not load the full theme from both locations. Duplicate registration makes
specificity and reload results misleading. Moving a stylesheet into `chrome/CSS`
also changes relative image paths: images left in `chrome/` would be referenced as
`../dark.png` and `../light.png`.

## Floorp MCP inspection status

Floorp 12.8 and newer can expose a loopback browser-automation API when
`floorp.mcp.enabled` is `true`. The official stdio bridge is
[`floorp-mcp-server`](https://github.com/Floorp-Projects/floorp-mcp-server), and
its documented generic client configuration runs `npx -y floorp-mcp-server`.
This capability is optional and is not part of Shina Fox's runtime.

For the Floorp 12.16.4 baseline, do not assume the published bridge can connect.
The audited npm release, `0.2.0`, creates its client with an empty token. Floorp
subsequently made bearer authentication mandatory in
[Floorp commit `4253743`](https://github.com/Floorp-Projects/Floorp/commit/42537437ffee85069e0a7af50d5f571582e4eeac),
and the current browser stores an owner-only token in
`~/.floorp/os-server-token`. Until the bridge reads that token or offers a safe
token input, it can initialize over MCP but current browser requests return
unauthorized. Never work around this by blanking or disabling the browser token.

Even after that bridge is updated, it controls tabs and logged-in page content;
start verification with the read-only `floorp_list_tabs` tool. It is not a
replacement for Firefox Browser Toolbox when inspecting browser-chrome DOM and
computed styles, which are what `userChrome.css` targets.

## Current browser UI map

Use Firefox Browser Toolbox against the installed build before introducing a new
selector. Prefer semantic classes and stable container IDs over anonymous markup
or generated inline styles. The Sidebery overlay is a deliberate exception: it
uses the signed add-on's stable ID so the same geometry is not applied to every
extension sidebar.

| Concern | Floorp 12 / Gecko 153 anchors | Migration note |
| --- | --- | --- |
| Main window state | `:root`, `#main-window`, `#navigator-toolbox` | Chrome fullscreen is exposed on the root as `[inFullscreen]`; page fullscreen uses `[inDOMFullscreen]`. The current tabs-hidden state is on `#navigator-toolbox`. |
| Address bar | `moz-urlbar#urlbar.urlbar`, `:popover-open`, `[breakout][breakout-extend]`, `.urlbar-background`, `.urlbar-input-container`, `.urlbar-input-box`, `.urlbar-input`, `.urlbar-go-button`, `.urlbarView-results` | The background, input container, and go button are classes now. `#urlbar-input` and `#urlbar-results` may still be assigned dynamically, but classes are the safer styling contract. The expanded urlbar is a `popover="manual"` top-layer element; geometry overrides must target that element and state, never a transformed toolbar ancestor. |
| Unified search switcher | `.searchmode-switcher-panel-list[open]`, `.searchmode-switcher[open]`, `.searchmode-switcher-panel` | Opening the switcher intentionally closes the normal urlbar results view, removing its expanded state. Floorp 12.16.4 sets `open` on the `panel-list` before dispatching its `showing` event, then sets `open` on the `moz-button` after the menu is shown. Observe both states with `#urlbar:has(...)` to bridge the complete transition; the non-popover urlbar needs viewport-fixed positioning during this state. |
| Native sidebar/vertical tabs | `#sidebar-container`, `sidebar-main`, `#vertical-tabs` | End-position state uses the presence attribute `[sidebar-positionend]`, not the old `[positionend]`. This is separate from the legacy extension sidebar. |
| Extension sidebar/Sidebery | `#sidebar-box`, `#sidebar-header`, `.sidebar-browser-stack`, `browser#sidebar` | `#sidebar-box` still hosts extension sidebars. The theme scopes rules to Sidebery's signed extension ID (`_3c078156-979c-498b-8990-85f7987dd929`); a self-built package must update that selector. |
| Floorp panel sidebar | `#panel-sidebar-box`, `#panel-sidebar-header`, `#panel-sidebar-browser-box`, `.sidebar-panel-browser`, `#panel-sidebar-splitter`, `#panel-sidebar-select-box` | This is Floorp's independent web-panel sidebar. Floorp already provides a floating mode; prefer its state/configuration over recreating floating geometry from a fixed width. |
| Page frame | `#browser`, `#tabbrowser-tabbox`, `#tabbrowser-tabpanels`, `.browserContainer`, `.browserStack` | `#appcontent` is gone. The theme frames `.browserContainer`, has a smaller split-view margin, and removes the frame in browser/DOM fullscreen. Any clipping used for rounded corners must stay on that container rather than the content `<browser>`. |

URL bar state selectors such as `#urlbar[breakout][breakout-extend]` still exist,
but Gecko now measures and places the `moz-urlbar` popover itself. The theme
leaves the idle state under Gecko's control and applies one narrowly scoped
`:popover-open` override to restore the viewport-centered expanded design. Keep
that width viewport-bounded and do not transform a toolbar ancestor: top-layer
elements do not inherit the clipping/positioning model of their visual ancestors.

The unified search switcher is a separate XUL panel nested under the urlbar.
Firefox closes the normal urlbar results view when this panel starts opening, so
`[breakout-extend]:popover-open` cannot describe the whole interaction. The theme
keeps the urlbar fixed to the same viewport coordinates from the early
`.searchmode-switcher-panel-list[open]` state through the later
`.searchmode-switcher[open]` button state. The preference
`browser.urlbar.scotchBonnet.enableOverride=false` is documented only as a
fallback for future incompatible markup, not as part of the supported baseline.

## Legacy selectors and fragile rules

The Firefox 123/Floorp 11-era stylesheet used several obsolete assumptions.
Some may remain temporarily during migration; once replaced, keep this table as a
regression map and do not reintroduce them:

| Legacy pattern | Problem now | Current direction |
| --- | --- | --- |
| `#appcontent` and `#appcontent browser` | Removed from current browser markup. | Use `.browserContainer`/`.browserStack` and the tabbrowser containers. |
| `#sidebar2-box`, `#sidebar2-header`, `#sidebar-splitter2`, `#sidebar-select-box`, `.browser-sidebar2` | Old Floorp panel-sidebar implementation. | Use the `#panel-sidebar-*` family and `.sidebar-panel-browser`. |
| `#urlbar-background`, `#urlbar-input-container`, `#urlbar-go-button` | These internals changed from IDs to classes. | Use `.urlbar-background`, `.urlbar-input-container`, and `.urlbar-go-button`. |
| `#navigator-toolbox[inFullscreen]` | Fullscreen state is no longer owned by this element. | Use `:root[inFullscreen]` and, where needed, `:root[inDOMFullscreen]`. |
| `#nav-bar[tabs-hidden]` | The state moved. | Use `#navigator-toolbox[tabs-hidden]`. |
| `[positionend]` | Replaced by a presence-only sidebar-position attribute. | Use `[sidebar-positionend]`. |
| Exact `[style="..."]` matches | Generated inline dimensions and child order vary with every layout/configuration. | Select the component/state, not a serialized style declaration. |
| Generic `[sidebarcommand$="-sidebar-action"]` geometry | Applies Sidebery's narrow overlay to every extension sidebar. | Use Sidebery's stable signed ID; document how differently signed builds can change it. |
| `#webextpanels-:hover` | Invalid/typoed selector that never matches. | Inspect and target the actual browser/window node. |
| Nested duplicate `#navigator-toolbox` fullscreen selector | Describes an impossible descendant and never matches. | Anchor window state at `:root`. |
| `clip-path: circle(80%)` on content browsers | Can crop pages, fullscreen media, focus rings, and popup surfaces. | Frame `.browserContainer`; if rounded corners require `overflow: clip`, remove that frame in all fullscreen states and regression-test dialogs and split view. |

Also audit custom properties before use. Older snapshots referenced
`--firefoxcss-top-bar-border-bottom-size`,
`--firefoxcss-top-bar-border-bottom-color`, `--firefoxcss-item-bg-color`, and
`--firefoxcss-urlbar-zoom-button` without defining them locally. Keep these names
defined while legacy sections use them, or migrate the declarations to project
tokens. A missing custom property without a fallback invalidates that declaration.

Gecko 152-era Floorp code also migrated toolbox/toolbar variables toward
`--toolbox-background-color` and `--toolbar-background-color`. During a transition,
use an explicit project token or a fallback chain instead of assuming Floorp's
compatibility shims will remain.

## Post-migration adjustments (2026-08-28)

After the initial Floorp 12.16.4 migration, two visual regressions were corrected without changing the intended design:

- **Urlbar focus border:** `moz-urlbar#urlbar[focused] .urlbar-background` previously fell back to Floorp's default teal `#329ca0` (`--toolbar-field-focus-border-color`). Added high-specificity override (`#main-window #urlbar[focused]`, `moz-urlbar#urlbar[focused]`) to unify border to `rgba(118,130,118,0.8)` and suppress `outline`/`box-shadow`, matching `:hover` / `[breakout][breakout-extend]` state. Split `.urlbar-background` background handling so `:not([breakout-extend])` remains `transparent` while `[breakout][breakout-extend]` (typing, expanded) restores solid `var(--toolbar-background-color)` (`#282927` dark / `#636363` light) with `5px` radius. Fixed stray `#z-index` typo in `#main-window #urlbar`. Resolved triple-border stacking (`#urlbar` outer `2px` + inner `.urlbar-background`/`.urlbar-input-container` `2px` each) by keeping outer `2px` only, inner `border:none`.

- **Top chrome dividers:** `#navigator-toolbox` `border-bottom: 1px solid rgba(130,130,130,0.3)` (via `--firefoxcss-top-bar-border-bottom-*` tokens) was valid after token definition and painted a horizontal 1px line between nav and browser plus a short vertical at top-left. Set tokens to `0px`/`transparent` and forced `#navigator-toolbox`, `#nav-bar`, `#TabsToolbar`, `#titlebar`, `toolbar` to `border:none`/`box-shadow:none`/`outline:none` on all edges.

No new visual design, no JS loader, no duplicate registration was introduced.

## Compatibility assumptions

The visual design currently assumes:

- Floorp's Lepton interface with a top navigation bar and horizontal tabs.
- Sidebery, when used, is opened in Firefox's extension sidebar (ID-scoped, no hover geometry).
- Floorp's panel sidebar is optional and distinct from Sidebery; rules work for both start and end positions.
- Firefox's native vertical-tabs/sidebar revamp may exist in DOM even with horizontal tabs — selectors must not style it.
- Floorp's built-in rounded-page effect should be disabled because this theme owns the page frame.

Floorp 12 was a substantial rewrite from Floorp 11, and Gecko UI markup continues
to change on the rapid release cadence. Compatibility work should target observed
state attributes and current components, while keeping narrowly scoped fallbacks
only when they are still testable.

## Upstream triage for this pass

The community fork had no pull request to merge. The closest upstream change was
[Shina-Fox PR #53](https://github.com/Shina-SG/Shina-Fox/pull/53), which addresses
sidebar behavior and ultrawide address-bar positioning. Its intent is accepted in
this compatibility pass, but the patch was reimplemented instead of copied:

- the upstream proposal fixes the URL bar to `50%` width and `left: 45%`, while
  this implementation targets only Gecko 153's open top-layer state and bounds
  its centered width against the viewport;
- it adds a permanent content offset tied to one sidebar width/aspect ratio,
  whereas this theme keeps the host narrow; and
- it still depends on pre-Floorp-12 layout assumptions.

The local implementation therefore constrains `#urlbar-container` responsively,
uses current `panel-sidebar-*` and `.browserContainer` anchors, and limits custom
popover geometry to the open enchanted-address-bar state. Upstream issues
[#48](https://github.com/Shina-SG/Shina-Fox/issues/48),
[#49](https://github.com/Shina-SG/Shina-Fox/issues/49),
[#50](https://github.com/Shina-SG/Shina-Fox/issues/50), and
[#51](https://github.com/Shina-SG/Shina-Fox/issues/51) remain useful regression
cases for ESR compatibility, page gaps, sidebar layering, URL overflow, and the
top navigation bar.

## Safe development and deployment

1. Find the intended profile through `about:support` or `about:profiles`; do not
   guess from directory names.
2. Work in this repository or a disposable test profile first. Repository files
   and profile files are independent unless a developer deliberately links them.
3. Keep root `userChrome.css` testing separate from Floorp live-loader experiments.
   Use a small, scoped file for live inspection rather than copying the monolith.
4. Open Browser Toolbox and confirm selectors against the running build. Test a
   rule with low impact before changing geometry or stacking contexts.
5. Run a syntax/diff check, then make a timestamped backup of the profile's current
   `chrome/` files.
6. Fully quit Floorp and verify the profile lock/process is gone before replacing
   root startup files. Never overwrite an active profile simply because no window
   is visible.
7. Deploy `userChrome.css`, `dark.png`, and `light.png` together, restart, and keep
   the backup available for rollback. A startup-loaded stylesheet cannot be fully
   validated with `Alt+R`.
8. If the UI becomes unusable, close the browser and temporarily rename
   `userChrome.css`; avoid modifying packaged application archives or enabling a
   privileged JavaScript loader to solve a CSS-only issue.

## Regression checklist

- [ ] Cold startup loads the root stylesheet with no CSS parse errors or unresolved
  required variables in Browser Toolbox.
- [ ] Navigation controls, horizontal tabs, Floorp's workspace widget, titlebar
  buttons, app menu, and overflow menu remain reachable at narrow and ultrawide
  window sizes.
- [ ] Normal, maximized, browser-fullscreen, and DOM-fullscreen states do not leave
  gaps, overlays, clipped video, or hidden window controls.
- [ ] The address bar works while idle, focused, typing, showing results, and showing
  permission/identity indicators; results stay on screen at multiple widths.
- [ ] Pinned/unpinned tabs, tab overflow, new-tab controls, and toolbar customization
  mode remain usable.
- [ ] Native sidebar/vertical-tab controls are not styled when disabled.
- [ ] Floorp's panel sidebar works closed, open, resized, and on either side.
- [ ] Page content, split views, internal pages, permission prompts, downloads, and
  fullscreen media are not clipped by the decorative browser frame.
- [ ] Bookmarks toolbar, context menus, extension buttons, and find bar remain usable.
- [ ] Light and dark schemes retain readable text, focus indicators, hover states,
  and sufficient contrast.
- [ ] Root startup loading and a scoped Floorp live-loader test are verified
  independently, with no duplicate copy of the theme active.

## Upstream implementation references

- [Firefox `userChrome.css` loader](https://searchfox.org/firefox-main/source/layout/style/GlobalStyleSheetCache.cpp)
- [Current Firefox browser-box markup](https://searchfox.org/firefox-main/source/browser/base/content/browser-box.inc.xhtml)
- [Floorp live CSS service](https://github.com/Floorp-Projects/Floorp/blob/main/browser-features/chrome/common/chrome-css/service.tsx)
- [Floorp live CSS entry classification](https://github.com/Floorp-Projects/Floorp/blob/main/browser-features/chrome/common/chrome-css/cssEntry.ts)
- [Floorp advanced UI customization](https://docs.floorp.app/docs/features/advanced-ui-customization/)
- [Floorp panel sidebar](https://docs.floorp.app/docs/features/panel-sidebar/)
