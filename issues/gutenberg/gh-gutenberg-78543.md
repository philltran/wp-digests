# #78543: Add support for layout responsive styles

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Layout`
- **Merged:** [`935d434`](https://github.com/WordPress/gutenberg/commit/935d434f0aff3e610e427e887936684eed217ab7)
- **Discussion:** [#78543](https://github.com/WordPress/gutenberg/pull/78543) · 4 comments · 1 reactions

## Summary

This PR adds editor support for per-breakpoint overrides of layout, block spacing, and child layout styles on blocks that use the block editor layout system. Editors can now toggle to a mobile state and assign a different column count, block gap, or child column span that applies below a breakpoint — without those overrides affecting the desktop layout. The change is part of the larger responsive layout initiative tracked in #77817. A known limitation at merge time: Navigation block responsive layout styles are not yet applied correctly.

## Impact

**Plugin & theme developers**
- No deprecated or removed APIs. The change is additive.
- The PHP layout style generation layer gains conditional output paths guarded by `shouldOutput*` logic. If you post-process or filter the CSS emitted by layout style functions, review whether those guards affect your output.
- The `@wordpress/block-editor` package grows by ~1.18 kB (minified JS).

**Site builders / content editors**
- Grid block: column count and block gap can now differ between desktop and mobile states.
- Grid children: column span can be set independently per breakpoint.
- Cover block: responsive rules apply correctly to its inner container layout.
- Buttons and other layout-enabled blocks: responsive styles are supported.
- Navigation block: responsive layout styles are **not** correctly applied yet — a follow-up is in progress.

**No action required** for sites that do not use responsive layout overrides; existing layout output is unchanged when no responsive values are set.

## Technical details

The only compiled artifact change is `build/scripts/block-editor/index.min.js` (+1.18 kB, +0.31%). The PR also modifies PHP-side layout style generation. Review discussion describes the PHP changes as introducing `shouldOutputX` boolean guards that control whether responsive `@media` rule blocks are emitted for layout, gap, and child layout styles — skipping output when no mobile-state overrides are defined, to avoid regressions on blocks that have no responsive values set.

The Cover block receives special handling because its layout styles are applied to an inner container element rather than the block's root element; the PR accounts for this distinction when generating responsive rules.

Child layout styles (e.g., `grid-column: span N`) are made responsive alongside parent layout styles, so a child block can declare both a default span and a mobile-state span independently.

No new `block.json` fields, REST schema changes, or options/constants are documented in the provided source material.

## Contribution

Opened and authored by @tellthemachines as part of the #77817 responsive layout project. @talldan is credited as a co-author via the props bot, indicating substantive review. Review feedback focused on the complexity of the PHP output-gating logic (`shouldOutputX` conditions); @tellthemachines addressed this feedback and simplified the PHP side before merge. The author disclosed use of GPT 5.5/Codex and Claude Opus 4.7 during development. Merged at commit `935d434` with Navigation block responsive support explicitly noted as a remaining known issue.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
