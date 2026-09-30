# #77141: Block Supports: Add background-clip block support infrastructure

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Blocks`, `[Package] Block editor`, `[Feature] Design Tools`, `[Package] Style Engine`
- **Merged:** [`6996db6`](https://github.com/WordPress/gutenberg/commit/6996db6ae7bcc7f1dd8dc8cba8753d1af01abbd1)
- **Discussion:** [#77141](https://github.com/WordPress/gutenberg/pull/77141) · 12 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

Adds a `background.backgroundClip` block support to the style engine, enabling `background-clip` values (`text`, `border-box`, `padding-box`, `content-box`) to be set via `theme.json` styles and block attributes. The `text` value also emits the `-webkit-background-clip` and `-webkit-text-fill-color` companions needed for cross-browser text-gradient rendering. No editor UI is included; that ships separately in #77142. The support is opt-in per theme via `settings.background.backgroundClip` and is not part of the appearance tools opt-in.

## Impact

- **Theme developers:** Can now declare `"backgroundClip": true` (or an array of allowed values) under `settings.background` in `theme.json`, and use `backgroundClip` in block-level `styles.background` to clip gradients or images to text or to a box. No action required for existing themes.
- **Plugin & block developers:** The new `backgroundClip` key is available in the `block.json` supports schema and the `theme.json` style schema. Blocks that declare `"supports": { "background": { "backgroundClip": true } }` will have the style engine emit the CSS.
- **Hosting / platform:** The three CSS properties (`background-clip`, `-webkit-background-clip`, `-webkit-text-fill-color`) are added to the kses allowlist in the 7.2 compat file. If you maintain a custom `safecss_filter_attr()` allowlist, you will need to add these three properties or the style engine output will be stripped.
- **No breaking changes.** Existing `theme.json` files and block styles are unaffected.

## Technical details

The support is registered under the `background` group in both style engines (the `@wordpress/style-engine` package and the PHP `WP_Theme_JSON_Resolver` path). The `theme.json` setting schema accepts `true`, `false`, or an array of allowed values.

**CSS emission for `text`:**

```css
background-clip: text;
-webkit-background-clip: text;
-webkit-text-fill-color: transparent;
```

**CSS emission for box values (`border-box`, `padding-box`, `content-box`):**

```css
background-clip: border-box;
-webkit-text-fill-color: currentColor;
```

The box values restore the fill colour with `currentColor` rather than `unset` because `-webkit-text-fill-color` is inherited; using `unset` would propagate a transparent fill from an ancestor that clips to text. The `-webkit-background-clip` property is intentionally *not* emitted for box values because in Chromium it is an alias of `background-clip`, and resetting it would discard the value just set.

**`has-background` class logic:** The class is added only when an image or gradient actually paints a background. It is never added when `backgroundClip` is `text`, in both the editor and the front end.

**kses allowlist:** The three properties are added to the 7.2 compat file so that `safecss_filter_attr()` does not strip them. Without this, the style engine would return empty CSS for any block using the support.

**Kebab-case handling:** Vendor-prefixed property names (e.g. `-webkit-background-clip`) are excluded from the kebab-case conversion pass, and custom properties (`--*`) are left untouched.

## Contribution

Opened by @aaronrobertshaw as a split from a larger PR; the author notes Claude Code was used to perform the split, rebase, and draft the description. The PR is part of the broader text-gradient work tracked in #30982 and #76525, with the editor UI deliberately deferred to #77142. The 12-comment discussion and single reaction in the provided record contain no visible design debate or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
