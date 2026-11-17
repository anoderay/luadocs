# dtdocs theme port — notes & build/verify checklist

Branch: `new-dtdocs-theme` (based on `upstream/development`).

This file is a scratch note for the port. It is **untracked** — delete it
(or leave it out of your commit) when you are done.

## What was done

The current dtdocs web theme (`darktable-org/dtdocs`,
`themes/hugo-darktable-docs-theme`) was ported onto luadocs, plus the two
requested customizations.

### Theme
- `themes/hugo-darktable-docs-theme/` was overlaid with the new dtdocs version
  (layouts, SASS, JS, i18n, static, archetypes). The vendored
  `assets/node_modules/` was **kept** (old bootstrap 4.5.3 / fork-awesome etc.
  satisfy the new theme's `package.json`, so a build without `yarn install`
  also works).
- The **version switcher** was removed (it is darktable-usermanual specific:
  it parses `/usermanual/<version>/` out of the baseURL and reads
  `site.Data/versions`, neither of which luadocs has). Removed from
  `baseof.html` and `page-header-navbar-large.html`.
- The unconditional **"Download manual (PDF)"** link in `page-meta.html` was
  removed (it pointed at `darktable_user_manual.pdf`, which luadocs has not).
- The **language switcher** was kept; it self-hides when the page has no
  translations.

### Blue nav dots (your request)
The six spectral "darktable logo" colors were replaced by a six-step navy-blue
ramp (light → dark, ending near Lua navy `#000080`). Changed in **both** places
that must stay in sync:
- `assets/sass/modules/variables.scss` → `$dt-logo-colors`
  (`#6ca8e0 #4a8bd4 #2f6ec4 #1f54aa #163f88 #0d2766`) — spectrum bar + nav dots.
- `layouts/partials/section-colors.html` → the same six colors as RGB triples —
  per-section accent colors.

### Lua logo in the header (your request)
- Official logo `static/lua-logo.gif` (128×128, from lua.org/images,
  **unmodified**) sits next to the darktable logo, on a light rounded "chip"
  (`.dt-lua-brand` in `header.scss`) so the navy artwork reads on the dark
  header. The chip is a container only; the logo itself is not altered.
- It **links to https://www.lua.org/** (satisfies lua.org's "please add a link"
  request) and carries a credit in its `title` and an HTML comment.
- A small **footer credit** was added (`page-footer.html`, re-enabled in
  `baseof.html`, styled as `.dt-footer` in `body.scss`): "The Lua logo is ©
  Lua.org, design by Alexandre Nakonechnyj…".
- License basis: lua.org/images allows the logo to refer to the Lua language,
  with a link back and without modifying the artwork.

### Config / build
- `config.yaml`: mounts `static/darktable.svg` and `static/lua-logo.gif` into
  `assets/images/` (the header reads them via `resources.Get "images/…"`);
  added `markup.tableOfContents`; added `params.editURL` for the
  "edit this page" link.
- `flake.nix`: added **`dart-sass`** — the new theme compiles its SCSS with the
  `dartsass` transpiler, which the old flake did not provide. **Without this the
  build fails.**

## Build & verify (nothing below was run — no toolchain in this container)

```bash
nix develop
cd themes/hugo-darktable-docs-theme/assets && yarn install && cd -
hugo server --config config.yaml -b http://localhost:1313/luadocs/
```

Then check in the browser:
1. Build completes — watch for Dart Sass / SCSS errors.
2. Header: darktable logo + Lua logo (on its light chip) + title; Lua logo
   links to lua.org; footer shows the Lua credit.
3. Left nav: top-level sections carry **blue** dots; the spectrum bar under the
   header is a blue gradient (no rainbow left anywhere).
4. No "Download manual (PDF)" link and no "Version:" dropdown.
5. `lychee` link check passes (CI step).

## Not done / out of scope
- The **epub/pdf themes** were not ported (separate outputs; `build-all.sh`
  currently builds HTML only — epub/pdf are commented out). Say the word to
  port those too.
- **Fira fonts**: the theme names Fira Sans/Fira Mono but loads them nowhere
  (dtdocs relies on the fonts being present); luadocs will fall back to system
  fonts. Add a web-font `@font-face`/Google Fonts link if you want the exact
  dtdocs look.
- Nothing was committed.
