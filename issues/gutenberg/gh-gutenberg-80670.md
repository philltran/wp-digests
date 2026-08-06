# #80670: Meta boxes: Match heading styles with Gutenberg panels

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @dilipom13
- **Labels:** `[Type] Bug`, `[Feature] Meta Boxes`, `[Package] Edit Post`, `First-time Contributor`
- **Merged:** [`fb1f9e1`](https://github.com/WordPress/gutenberg/commit/fb1f9e1036ff66c82e366675c9b0bb990499d17c)
- **Discussion:** [#80670](https://github.com/WordPress/gutenberg/pull/80670) · 7 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

This pull request updates the visual styling of PHP meta box headings in the block editor to match the typography and border styles of native Gutenberg sidebar panels. Previously, meta box titles used inherited colors and lighter borders, creating a visual inconsistency with the rest of the editor UI. The change standardizes heading colors, font sizes, and top borders across all meta boxes, improving visual cohesion without altering functionality or interaction patterns.

## Impact

- **Plugin & theme developers:** No code changes required. The update only affects CSS rendering in the block editor.
- **Site owners & editors:** Meta box titles in the editor will now appear darker and consistent with sidebar panels, improving readability and visual alignment.
- **No action required.** This is a purely cosmetic UI adjustment with no breaking changes, deprecations, or migration steps.

## Technical details

The change modifies `packages/edit-post/src/components/meta-boxes/meta-boxes-area/style.scss` to align meta box header styles with the editor's design system. Specifically:
- `.postbox-header` border-top color is updated from `$gray-300` to `$gray-200`.
- Selectors `#poststuff .stuffbox > h3` and `#poststuff h2.hndle` now explicitly set `color: $gray-900` and `font-size: $default-font-size`, replacing the previous `color: inherit` rule.
```diff
-		color: inherit;
+		color: $gray-900;
+		font-size: $default-font-size;
```
The diff does not introduce new hooks, filters, or JavaScript logic; it is a scoped CSS update targeting legacy PHP meta box wrappers within the block editor context.

## Contribution

Opened by @dilipom13 to address visual inconsistency tracked in issue #12101. Reviewer @jasmussen requested before/after screenshots to validate the visual change, which were provided and approved. The PR moved quickly through review with a design gut-check from the Gutenberg team, and was merged without significant debate or alternative approaches.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
