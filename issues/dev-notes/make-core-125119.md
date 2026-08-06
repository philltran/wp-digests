# Pseudo and custom style states in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Daniel Richards
- **Published:** 2026-08-05
- **Tags:** `General`, `7.1`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/08/05/pseudo-and-custom-style-states-in-wordpress-7-1/](https://make.wordpress.org/core/2026/08/05/pseudo-and-custom-style-states-in-wordpress-7-1/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 introduces native support for styling pseudo states (`:hover`, `:focus`, `:focus-visible`, `:active`) and custom states (`-current`) directly in `theme.json` and block instance attributes. This allows theme authors and site editors to define state-specific styles for blocks like Button and Navigation Link without writing custom CSS. The feature also adds an editor setting to control whether users can edit these states in the block inspector and Global Styles UI.

## Impact

- **Theme & plugin developers:** Can now define state styles declaratively in `theme.json` using `:`-prefixed pseudo-state keys and `-`-prefixed custom-state keys under block styles. Existing state styles written via custom CSS or PHP filters remain unaffected.
- **Site editors:** Gain access to state dropdowns in the block inspector and Global Styles UI (unless disabled).
- **Hosting & platform teams:** No action required. Existing content renders exactly as before, and state styles already saved in `theme.json` or block attributes are preserved.
- **Opt-out configuration:** Sites can disable the UI for editing block states by setting `blockStatesEditingEnabled` to `false` via the `block_editor_settings_all` filter.

## Technical details

The style engine now parses `:`-prefixed pseudo-state keys and `-`-prefixed custom-state keys inside the `styles.blocks` object of `theme.json`. These keys can be nested within responsive state keys like `@mobile`. Block instance attributes support the identical structure inside the `style` property. Custom states map to CSS selectors defined in a block’s `block.json` `selectors.states` property; for example, `core/navigation-link` defines `"-current": ".wp-block-navigation-link .current-menu-item"`, which the engine uses to generate the final CSS selector. The editor UI is controlled by the `blockStatesEditingEnabled` setting (defaults to `true`). Setting it to `false` via `add_filter( 'block_editor_settings_all', 'fn' )` hides the state dropdowns in the block inspector and Global Styles, but does not strip already-saved state styles. Viewport state editing remains separate and is controlled by `responsiveEditingEnabled`.

## Contribution

The record carries no discussion detail beyond the author and reviewers, but the implementation ships declarative `theme.json` support first, with UI controls gated behind `blockStatesEditingEnabled` to allow gradual rollout.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
