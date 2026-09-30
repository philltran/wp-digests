# What’s new in Gutenberg 24.1? (30 September)

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Copons
- **Published:** 2026-09-30
- **Tags:** `General`, `block-editor`, `core-editor`, `gutenberg`, `gutenberg-new`
- **Link:** [https://make.wordpress.org/core/2026/09/30/whats-new-in-gutenberg-24-1-30-september/](https://make.wordpress.org/core/2026/09/30/whats-new-in-gutenberg-24-1-30-september/)
- **Usefulness:** 4/5

## Summary

Gutenberg 24.1 is a design-tool-consistency release (~140 PRs) that extends background image/size/gradient, shadow, border, link/heading/button color, typography and text-columns supports to nearly every core block. It adds a `textShadow` typography support with its own Global Styles panel (Paragraph and Heading first), a public API for viewport states in custom controls, and a Media section in the Cover and Media & Text inspector panels. It also stabilizes a few experimental APIs, deprecates several `@wordpress/components` primitives, removes the Fonts API BC layer, and raises the plugin's minimum WordPress version to 7.0.

## Impact

**Theme developers / block authors**
- Many core blocks now accept `theme.json` and `block.json`-driven background, shadow, border, colour and typography settings they did not before. Audit existing themes: new controls may appear in the editor, and `theme.json` styles for these blocks can now take effect.
- `typography.textShadow` is a new block support (#79584), available on Paragraph and Heading, with a Global Styles panel.
- Viewport states now have a public API for custom controls (#82741), and position can be set per viewport (#83094).
- Stabilized names: `unstableFeaturedImageFlow` → `featuredImageFlow` (#83439) and `__unstableNormalizeArgs` → `normalizeArgs` (#83067). Migrate off the `unstable`/`__unstable` names.
- The `editor.PostFeaturedImage` filter is now supported in the post summary featured image field (#83133).
- Deprecated components: `Divider`, `ResponsiveWrapper`, `Scrollable`, `ZStack`, `Elevation`. The deprecated `__experimentalGroup` prop was removed (#83171).
- The Video block no longer handles the GIF variation (#82696). The Icon block no longer renders non-public icons on the frontend (#82774).
- New `@wordpress/ui` primitives: `CheckboxGroup`, `RadioGroup`, `RadioGroupControl`, `Switch`, `SwitchControl`.

**Plugin / hosting**
- The Gutenberg plugin now requires WordPress 7.0 or newer (#82404).
- The deprecated Fonts API BC layer was removed (#82813). Check for reliance on it.
- Scripts tooling: unit tests moved to Vitest and Jest tooling deprecated (#82843).

**Site owners**
- No action required beyond visual QA. Fixes include Navigation submenu behavior, Cover in Safari, Image block link colour leakage, meta boxes hidden in Code Editor view, and HEIC upload detection.

## Technical details

This is a release roundup, so details come from the changelog entries rather than diffs.

- **Block supports:** Roughly 100 PRs are per-block enablement of existing supports (background image/size/gradient, `shadow`, `border`, link/heading/button colour, `typography`, text columns, text align). Group, Post Author Biography and Terms List gain text columns (#83425, #83424, #83432). Tabs gains typography (#83431), and Tab Panel and Tab Panels gain typography (#83426, #83428).
- **Text shadow:** Block API `textShadow` typography support and UI (#79584), building on the `theme.json` text shadow introduced in 23.5 (#73320).
- **Media inspector:** Cover (#82751) and Media & Text (#82593) get a media selection section in the inspector, matching Image and Site Logo.
- **Data layer:** `getResolutionArgs` added to share a resolution across selector calls (#82638). `__unstableNormalizeArgs` is stabilized as `normalizeArgs` (#83067). Async-mode `useSelect` notification is deferred to idle time (#82821) and store listeners are optimized (#82842).
- **Icons:** New `core-admin` collection (#83261). The `public` property moves from icons to collections (#83277), so icons can ship to Core without appearing in the Icon block (#82634, #79451). The registry sanitizer now allows `rect` and `circle` (#82846). The stroke redraw is completed (#82540, #82754).
- **PHP-only blocks:** Enums in array attributes are now detected (#83394). The Pattern block runs `autoembed()` before `do_blocks()` (#82860). `WP_Block_Parser` methods use the null coalescing operator (#82763).
- **Fixes:** Image block no longer interprets `$` and `\` in `img` attributes as regex backreferences (#79369). Term Name now applies term name display filters (#82365). Media detects HEIC by file header (#81737) and no longer injects `crossorigin` under Document-Isolation-Policy (#82614).
- **Other:** Dependency Extraction Webpack Plugin now bundles `global-styles-engine` and `global-styles-ui` (#82589) and pretty-prints its asset output (#79650). React is upgraded to 19.3.0 (#82760), and Ariakit and Base UI are bumped.

## Contribution

The post is the biweekly release summary by @Copons and does not record design debate. The support-expansion work was split into many small per-block PRs, which accounts for the ~140-PR count. Seven first-time contributors landed PRs in this release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
