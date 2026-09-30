# #82024: Block Library: Allow themes to override the padding added via background color

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] List`, `[Block] Heading`, `[Block] Columns`, `[Block] Paragraph`, `[Block] Preformatted`, `[Block] Group`, `[Feature] Design Tools`, `[Block] Template Part`, `[Package] Base styles`
- **Merged:** [`3516e59`](https://github.com/WordPress/gutenberg/commit/3516e59e2bf8d484800b0ad0ea6dfcb0bd6c31f2)
- **Discussion:** [#82024](https://github.com/WordPress/gutenberg/pull/82024) · 8 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Seven core blocks (Paragraph, List, Heading, Preformatted, Columns, Group, Template Part) that add default padding when a background color is set now read that padding from a new CSS custom property, `--wp--style--block-background-padding`. The existing values (`1.25em 2.375em`) remain the fallback, so nothing changes unless a theme sets the property. Themes can now change or remove the padding, including on Heading where it previously could not be overridden at all.

## Impact

- **Theme developers**
  - Opt-out or override is now possible by setting `--wp--style--block-background-padding` to any valid `padding` shorthand (e.g. `0`, `1rem 2rem`).
  - Setting it on `body` applies globally; setting it via block-level custom CSS (e.g. `styles.blocks.core/group.css`) scopes it to that block type, and nested blocks inherit it unless they set their own.
  - No `theme.json` settings key exists for this; you need `styles.css` or block-level custom CSS.
- **Site owners / existing sites**: No action required. Rendering is unchanged when the property is unset.
- **Plugin / SCSS authors using `@wordpress/base-styles`**: A new `$block-bg-padding` variable is available. `$block-bg-padding--v` and `$block-bg-padding--h` still exist but are now only the fallback values, and referencing them directly bypasses the theme override.
- **Headless / REST consumers**: Not affected.

## Technical details

The diff adds a Sass variable in `packages/base-styles/_variables.scss`:

```scss
$block-bg-padding: var(--wp--style--block-background-padding, #{$block-bg-padding--v} #{$block-bg-padding--h});
```

Each block stylesheet replaces `padding: $block-bg-padding--v $block-bg-padding--h;` with `padding: $block-bg-padding;`. Files touched: `columns/style.scss`, `group/theme.scss`, `heading/style.scss`, `list/style.scss`, `paragraph/style.scss`, `preformatted/style.scss`, `template-part/theme.scss`. Selectors and specificity are untouched (still `:where(...)` / `:root :where(...)` forms), which is why existing output stays equivalent. The property is resolved inside the rule, so no selector changes are needed to override it.

Group and Template Part load these styles from `theme.scss`, which only loads when a theme calls `add_theme_support( 'wp-block-styles' )`.

Example opt-out in `theme.json`:

```json
{
  "version": 3,
  "styles": {
    "css": "body { --wp--style--block-background-padding: 0; }"
  }
}
```

Compiled CSS grows by roughly 700 B total across the affected stylesheets. Changelog entries were added to `base-styles` and `block-library`.

## Contribution

@aaronrobertshaw opened the PR (following #36586) and got reviews from @ramonjd, @andrewserong and @jasmussen. Joen noted an opt-in would be preferable, but Aaron argued opt-out is the cleanest for backward compatibility, since what matters is the version a theme or its content was built for rather than the Gutenberg version. Aaron also prototyped a `theme.json` setting for this property in #82057 and held this PR to land them together, but later closed that PR because its benefit was debatable, then rebased and merged this one alone.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
