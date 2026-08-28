<div align="center">

# Shina Fox - Make Firefox a Cozy Home
<a href="https://discord.gg/txgKQUvwnY">Shina's Discord</a>

Welcome to Shina Fox, a project that seeks to transform your Firefox experience
into something homely and comfortable. It focuses on making the browser feel
minimal, cozy, and fun to use. Enjoy!

[Original project](https://github.com/Shina-SG/Shina-Fox) ·

</div>

![](media/t1jwYxBFR7SimysiWYN5ZQmdgKSREYETuzEXhMXHdB9NC02gw6ghjRp4twz1Oryt9gCkmkHVqUvg7M7yGNfp3MreQrlYa6QcLrllA5wJIZaocdStB54gTQgujVdXBoFocvYUubOxLomxQwvrRhhZH4kzDANTPGM2spoUSymrRo7BheOuTOgMMgSACLRjCCUNUd3KlVqr9yFcloiCSgzZQ0LssMpm5URnUPwk0278M7nkwcLVYYJUJPxmyK.png)
![](media/IIXLiV9iZrsHJlRzcWqpOQPu5nTI2kVheTz6iZ8ynnwJb8mc2Xdyrc8d6zkpbRRn6a4OEdL02tai6EIM0mzgbxWNmKjv3zY5oJUvGHddEl8uwE7RVPYOKUsfphA26KONO3lw30RpqHBpO24kbP1RooZDG7IRRjf6yrlrOOnBKIf3dFmfIr5q5FptuF6QOiSgmOP8a3FrmnLWM8SAIoB9Qq1cd93rhPLYVw3aXq6VhJuPpMxnhLEuYkDQf7.png)

## Set-up Guide 🛠️

##### Step 1: Sidebery Configuration 🦔

- Firstly, install the [Sidebery Addon](https://addons.mozilla.org/firefox/addon/sidebery/).
- Navigate to Sidebery's `Settings` ⚙️ --> `Navigation bar` 📍.
- Turn off the `Show navigation bar in one line` option.
- Proceed to Sidebery's `Settings` ⚙️ --> `Style editor` 🎨.
- Finally, copy and paste the following CSS code snippet into the provided space:

```css
#root.root {--tabs-font: 0.9rem sans-serif;}
#root.root {--tabs-count-font: 0.5625rem sans-serif;}
#root.root {--nav-btn-margin: 1.2px;}
#root.root {--nav-btn-width: 26px;}
#root.root {--nav-btn-height: 26px;}

#root.root {--tabs-indent: 12px;}
#root.root {--ctx-menu-font: 1rem Monospace;}
#root.root {--ctx-menu-min-width: 256px;}
#root.root {--ctx-menu-max-width: 9999px;}

/* unloaded tab title style */
.Tab[data-discarded="true"] .title {
    opacity: 80%;
}

/* adjustments */
#root.root {--tabs-indent: 16px;}
#root.root {--tabs-audio-btn-width: 32px;}
#root.root {--tabs-inner-gap: 4px;}

/* indent indicator ********************************************/
/* Settings > Tabs > Show marks to indicate tabs sub-tree levels > on */
.Tab .title {
    transition: margin-left 300ms ease;
}
.Tab:hover {
    --tabs-inner-gap: 8px;
}
.Tab:hover .title {
    margin-left: 8px;
}
```

The Style editor changes Sidebery itself. Shina Fox's `userChrome.css` changes
the Firefox/Floorp frame that contains it, so both parts are needed for the
intended result.

##### Step 2: Firefox & Floorp Configuration ⚙️

- In the URL bar, enter `about:config` and select `Accept the Risk and Continue`.
- Verify the following preference:

| Configuration Parameter 🛠️ | Required Setting 🎛️ |
| --- | --- |
| `toolkit.legacyUserProfileCustomizations.stylesheets` | `true` |

- In Floorp, open `about:hub#/features/design` and keep the navigation bar at the
  top. Disable Floorp's built-in **Round corners of pages**, because Shina Fox
  supplies its own framed page design.
- In **Floorp Hub → Tabs & Appearance → UI Customization → Bookmark Bar**, leave
  **Expand bookmark bar on focus** disabled. Shina Fox reveals bookmarks only
  when the URL capsule is hovered.
- Right-click an empty toolbar area and set **Bookmarks Toolbar → Always Show**.
  The toolbar stays visually folded until it is needed.
- If the unified search button changes the URL bar position, see the
  [Troubleshooting](#troubleshooting) section below.

##### Step 3: Theme Selection 🎨

- In the URL bar, type `about:addons`.
- Navigate to the `Themes` section and select Dark, Light, or another preferred
  [theme](https://addons.mozilla.org/firefox/themes/).
- A fixed light or dark theme is the easiest baseline while troubleshooting.

##### Step 4: File Installation 🗂️

- Download the
  [Latest commit](https://github.com/AbrarAbe/Shina-Fox/archive/refs/heads/main.zip)
  and extract it outside the browser profile.
- Enter `about:profiles` in the URL bar.
- Find the profile currently in use and click `Open Directory` beside its
  `Root Directory`.
- Fully quit Firefox/Floorp and make a backup of any existing `chrome` folder.
- Create a lowercase `chrome` folder if one does not already exist.
- Copy these files directly into it:

```text
<active-profile>/
└── chrome/
    ├── userChrome.css
```

The exact stylesheet name is `userChrome.css`, not `chrome.css` or
`userChrome.js`.

> [!IMPORTANT]
> Shina Fox uses the standard startup file at `chrome/userChrome.css`. Do not
> duplicate the whole theme inside Floorp's live-CSS directory at `chrome/CSS/`.
> The standard file requires a full browser restart; `Alt+R` reloads only the
> separate live-CSS directory.

Restart Firefox/Floorp to experience your new cozy haven! 💓🎉

## Feature Breakdown 🌟

| Feature Name 🌈 | Preview 📸 |
| --- | --- |
| Adaptive Theme | ![](media/Adaptive%20Theme.gif) |
| MacOS Buttons | ![](media/MacOS%20button.gif) |
| Highlight Border | ![](media/Highlight%20Border.gif) |
| Minimal Extension Menu | ![](media/Minimal%20Extension%20Menu.gif) |
| Enchanted URL Bar | ![](media/Enchanted%20URL%20Bar.gif) |
| Custom Icon | ![](media/Custom%20Icon.gif) |
| Hovering Bookmark | ![](media/Hovering%20Bookmark.gif) |

<details id="troubleshooting">
<summary>Troubleshooting & common fixes 🧰</summary>

- **Nothing changes:** Confirm that you edited the active profile, the file is
  exactly `chrome/userChrome.css`, and
  `toolkit.legacyUserProfileCustomizations.stylesheets` is `true`. Restart the
  browser after changing the file.
- **The unified search button makes the URL bar jump:** The current stylesheet
  keeps the menu centered. On an older Floorp build, try setting
  `browser.urlbar.scotchBonnet.enableOverride` to `false` in `about:config`,
  then restart. This workaround disables the grouped Scotch Bonnet features, so
  restore the default after updating Floorp or Shina Fox.
- **Bookmarks do not appear on hover:** Set **Bookmarks Toolbar → Always Show**,
  keep **Expand bookmark bar on focus** disabled in Floorp Hub, and hover the URL
  capsule itself rather than the whole navigation bar.
- **Rounded pages or panel spacing look wrong:** Disable Floorp's built-in
  **Round corners of pages** and configure the Panel Sidebar separately from
  Sidebery.

For selector details and regression checks, see the
[architecture notes](docs/ARCHITECTURE.md).

</details>

## Appreciation 🌟:

Shina Fox was created by **Shina-SG**. The visual identity, original stylesheet,
previews, project name, and welcoming style of this guide come from the
[original Shina Fox repository](https://github.com/Shina-SG/Shina-Fox). This
patch exists to preserve that work and keep it usable as Firefox and
Floorp evolve.

The original project used code and inspiration from the community. Please check
out their incredible work:

| Credits 📝 |
| --- |
| [Shina-SG - Original Shina Fox creator](https://github.com/Shina-SG) |
| [Community Edition repository](https://github.com/SetyVII/Shina-Fox-Comunnity-Edition).
| [Firefox Mod Blur - datguypiko](https://github.com/datguypiko/Firefox-Mod-Blur) |
| [Firefox-ONE - Godiesc](https://github.com/Godiesc/firefox-one) |
| [Floorp Projects](https://github.com/Floorp-Projects/Floorp) |
| [Mozilla Firefox](https://hg.mozilla.org/mozilla-central/) |
| The FirefoxCSS, Floorp, and Sidebery communities |

## Patch Edition & License 🦊

This repository is a continuation, not a replacement of the original
creator or an official Floorp/Mozilla product. Original work remains credited to
Shina-SG

Shina Fox and this patch are distributed under the
[Mozilla Public License 2.0](LICENSE). Please preserve the license and
attribution when redistributing modified versions.