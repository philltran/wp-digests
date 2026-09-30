# #79401: Image Block: Prevent global link colors from leaking into linked image borders

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @dpmehta
- **Labels:** `[Type] Bug`, `[Package] Block library`
- **Merged:** [`eda94fb`](https://github.com/WordPress/gutenberg/commit/eda94fbd2e28efa0487e51019ffebabfa928165a)
- **Discussion:** [#79401](https://github.com/WordPress/gutenberg/pull/79401) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Image block's front-end stylesheet now sets `color: inherit` on anchors that directly wrap the image (`.wp-block-image > a` and `.wp-block-image > figure > a`). Previously, a linked image with a custom border width but no explicit border color would pick up the theme's global link color (or the browser default blue) as its border, because the border defaults to `currentColor`. With the fix, the border follows the surrounding block text color. The same applies to images in a Gallery block linked to the media file.

## Impact

**Site owners / theme authors**
- Linked images with a custom border and no explicit border color will render the border in the inherited text color instead of the global link color. This is a visible change on existing sites that had this combination.
- Themes that intentionally relied on the link color showing through on a linked image's border (via `styles.elements.link` in theme.json) will see the border change. To restore it, set an explicit border color on the block or add a more specific CSS rule.
- Because the rule sets `color` on the anchor, any theme CSS that relied on the link color cascading through the anchor to its children will also be affected, though the diff only targets anchors that are direct children of `.wp-block-image` or its `figure`.

**Plugin developers**
- No API, hook, or markup changes. No action required beyond visual regression checks if you style linked images.

**Design question raised in review**
- A reviewer (@carolinan) questioned whether overriding a site-wide link color for these anchors matches theme designers' expectations. @youknowriad supported the change on the grounds that links wrapping images are not semantically "link" elements. The PR was merged.

## Technical details

The diff is a single added declaration in `packages/block-library/src/image/style.scss`:

```scss
.wp-block-image {
	> a,
	> figure > a {
		display: inline-block;
		color: inherit; // new
	}
}
```

The underlying cause is that the `<a>` picks up the global link color from theme.json element styles, `color` is an inherited property, and an `<img>` border with no `border-color` resolves to `currentColor`. Setting `color: inherit` on the anchor makes it take the color of the parent `.wp-block-image`.

Per the PR description, an alternative was tried and rejected: excluding image anchors from the global `ELEMENTS` link selector with `:not(:has(> img))`. That was insufficient because the anchor then falls back to the browser's default link color, which still colors the border.

No PHP, theme.json, block.json, or REST changes.

## Contribution

The change came from @dpmehta and closes issue #53070, an older report. Review discussion centered on design intent: @carolinan questioned whether overriding a site-wide link color was what theme authors would expect, while @youknowriad agreed with the change, arguing that links wrapping images don't correspond to the "link" element. The PR also initially lacked a `[Type]` label, which was added before merge. The author noted AI tools helped draft the analysis and description.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
