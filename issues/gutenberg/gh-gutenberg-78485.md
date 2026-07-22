# #78485: Search block: Add opt-in support for the semantic <search> element

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Search`
- **Merged:** [`9602efe`](https://github.com/WordPress/gutenberg/commit/9602efe83831df5d38d8320a6cc9f7a3f0342826)
- **Discussion:** [#78485](https://github.com/WordPress/gutenberg/pull/78485) · 20 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `core/search` block now supports rendering inside the semantic `<search>` HTML element instead of `<form role="search">`. This opt-in capability preserves backward compatibility with existing theme CSS and layout patterns while allowing developers to adopt native landmark semantics. The feature is controlled via a new `tagName` block attribute and respects the `search-element` HTML5 theme support flag.

## Impact

- **Theme & plugin developers:** Sites can opt in globally by calling `add_theme_support( 'html5', array( 'search-element' ) )`, or per-block via the new Advanced inspector selector.
- **CSS & layout authors:** When opted in, the block outputs a `<search>` wrapper and drops `role="search"` from the inner form. Selectors like `form[role="search"]` or direct-child flex/grid layouts targeting the block root will no longer match and require updates.
- **Site owners:** No action required; default output remains unchanged.

## Technical details

The change introduces a `tagName` string attribute (default `""`) to `packages/block-library/src/search/block.json`. In `packages/block-library/src/search/edit.js`, a `SelectControl` inside an `<InspectorControls group="advanced">` panel maps `''` to "Default", `'search'` to `<search>`, and `'form'` to `<form>`, dynamically swapping the editor's wrapper element. The PHP renderer in `packages/block-library/src/search/index.php` evaluates `$attributes['tagName']` and falls back to `current_theme_supports( 'html5', 'search-element' )` when empty. It uses positional `sprintf` placeholders (`%1$s` for the action URL, `%2$s`/`%3$s` for wrapper attributes/directives) to maintain correct markup ordering across both wrapper types. The `<search>` variant applies block wrapper attributes and Interactivity API directives to the outer landmark, preserving alignment, spacing, and expandable-searchfield state classes.

## Contribution

Opened by @adamsilverstein and merged with co-authors @westonruter, @joedolson, @carolinan, and @mamaduka. Early review from @carolinan recommended aligning with the existing HTML element selector pattern used in other blocks rather than a custom toggle, which the author implemented. @westonruter’s feedback on potential CSS selector breakage reinforced the decision to keep the feature opt-in and defer to theme support.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
