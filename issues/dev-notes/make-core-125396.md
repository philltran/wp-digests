# What’s new in Gutenberg 23.7? (05 August)

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Jonathan Bossenger
- **Published:** 2026-08-06
- **Tags:** `General`, `block-editor`, `core-editor`, `gutenberg`, `gutenberg-new`
- **Link:** [https://make.wordpress.org/core/2026/08/06/whats-new-in-gutenberg-23-7-05-august/](https://make.wordpress.org/core/2026/08/06/whats-new-in-gutenberg-23-7-05-august/)
- **Usefulness:** 4/5

## Summary

Gutenberg 23.7 is a consolidation release: most of the cycle went into visual editor fixes rather than new surface area. The Gallery block's dynamic mode — new in 23.6 — gets its ambiguous "Convert to images" toolbar button renamed to "Detach" with an explanatory modal (#80727), and dynamic-mode conversion now collapses into a single undo level (#80665). The experimental Global Styles inheritance UI, which reflects inherited values in block inspector controls, has been moved behind an opt-in toggle on the Gutenberg Experiments screen (#80815). Alongside that: a new `responsiveEditingEnabled` editor setting, `editableRoot` demoted from a public block support to a private block setting, and `Group`/`GroupLabel` subcomponents added across Combobox, Select and SelectControl.

## Impact

**Block & plugin developers**
- `editableRoot` is now a **private block setting rather than a public block support** (#80820). If you set it on a custom block expecting supported public-API behavior, stop relying on it — it is no longer part of the documented supports surface.
- New editor setting `responsiveEditingEnabled` (#80814) lets you hide the Responsive styles option from the post editor. Opt-in behavior; no action if you don't need it.
- `Combobox`, `Select` and `SelectControl` gain `Group` and `GroupLabel` subcomponents (#80574), so grouped options now compose the same way across the item-popup families.
- Widget Primitives now ship as a **script module** (#80149) rather than a classic script — relevant if you enqueue or depend on them.
- Several components dropped `__next40pxDefaultSize` props from form controls (#80540) and the ESLint denylist gained more not-recommended components (#80754); expect new lint warnings on upgrade.

**Theme developers**
- Block-level preset classes generated from `theme.json` are now wrapped in `:where()` to level specificity (#80657). Custom CSS that previously lost (or won) a specificity battle against those preset classes may resolve differently — worth a visual regression pass on themes with hand-written overrides of preset color/typography classes.

**Editor / a11y users**
- The Math block no longer fires LaTeX parse errors on every keystroke; validation is deferred to first blur and the per-keystroke screen reader announcement is gone (#80500).
- Fixed device preview heights for mobile and tablet views are restored (#80271).

**Contributors / build**
- Jest updated to **v30, flagged breaking** (#80767). `DependencyExtractionWebpackPlugin` now includes extracted styles in the asset version hash (#80601), so cache-busting hashes will change.

For most site owners running Gutenberg as a plugin, no action is required.

## Technical details

**Gallery dynamic mode.** The toolbar action that turns a dynamic (query-driven) gallery into static images is relabeled from "Convert to images" to "Detach" and now opens a modal describing what detaching does (#80727). The conversion is registered as one undo step instead of several (#80665).

**Global Styles inheritance.** #77894 landed the work to reflect inherited Global Styles values in block inspector controls; #80815 gates the whole inheritance UI behind a Gutenberg experiment, so it is off unless enabled on the Experiments settings page. Related: #80607 resolves link element styles in block inspector controls for blocks that are links, and #80497 fixes the wrong preset being committed and displayed when two color presets share the same hex value (with the equivalent identity fix for gradients by slug in #80554).

**Block API surface.** `editableRoot` moves out of public block supports into a private block setting (#80820). A new `responsiveEditingEnabled` editor setting hides the Responsive styles option (#80814).

**Server-side PHP.** `render_block_core_video()` now accounts for a non-lowercase `VIDEO` tag when parsing block output (#80537). The Media REST API backports the upload size check for sideloading from a URL path (#80659). `WP_Sync_Config` switches to `str_contains()` (#80476), the Table of Contents block does the same (#80418), and the Page List block uses null coalescing for the `parentPageID` attribute (#80728). Real-time collaboration code moved to `lib/experimental/collaboration` (#80469).

**View configuration API.** Versioning was updated (#80319) and merge semantics tightened: shape-mismatched merges are rejected, empty-array semantics are defined, and nulls are stripped from appended members (#80571). Documentation now requires server-side view configuration filter callbacks to return the container (#80642, #80744), with new end-to-end coverage for view configuration extensibility in the site editor (#80577).

**Interactivity API.** Directives were refactored into self-registering modules (#79975).

**Robustness fixes.** `createInterpolateElement` handles unmatched closing tags gracefully instead of throwing (#80618); RichText no longer crashes in the paste handler when files are pasted (#81010) and skips input transforms when there is no `onReplace` (#80978). Dashboard widget action and help link `href`s are now sanitized (#80510, #80409).

**Theme JSON output.** Block-level preset class specificity is leveled with `:where()` (#80657).

## Contribution

This is the routine biweekly release roundup, so it carries changelog and credits rather than design debate. Three first-time contributors landed PRs in this cycle — @dilipom13 (meta box heading styles), @faisalahammad (restoring fixed device preview heights) and @nickchomey (the Interactivity API directive refactor). The post credits @mamaduka for review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
