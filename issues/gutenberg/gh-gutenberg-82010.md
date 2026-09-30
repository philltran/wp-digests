# #82010: Font Library: Keep CSS custom properties unquoted in font previews

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Feature] Font Library`
- **Merged:** [`821f1cb`](https://github.com/WordPress/gutenberg/commit/821f1cb51e23a1213937b82206d4a7375c808780)
- **Discussion:** [#82010](https://github.com/WordPress/gutenberg/pull/82010) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Font Library's `formatFontFamily` helper in `@wordpress/global-styles-ui` no longer wraps a bare `var(--custom-property)` value in quotes. Previously a theme whose `theme.json` font family was set to something like `var(--wp--preset--font-family--body)` produced `font-family: "var(--wp--preset--font-family--body)"`, which the browser treats as a font name, so the preview rendered in the fallback font. The value is now left as a CSS function call and the preview renders in the themed font.

## Impact

- **Theme developers:** Font families defined in `theme.json` as `var(--...)` references now preview correctly in Site Editor → Styles → Typography → Fonts. No theme changes are required.
- **Site owners:** Font previews for such themes show the real font instead of the fallback. Other previews (Styles → Typography, the font family list, the Font Library modal) are stated to be unchanged.
- **Limitation:** Only a bare `var(--name)` is handled. `var(--name, fallback)` is still quoted (see the TODO in the code).
- No action required; this is a preview-only fix with no public API change.

## Technical details

The change is in `packages/global-styles-ui/src/font-library/utils/preview-styles.ts`. `formatFontFamily` splits the input into items and quotes any item matching a regex that means "needs quoting". The regex gains a negative lookahead:

```diff
-const regex = /^(?!generic\([ a-zA-Z\-]+\)$)(?!^[a-zA-Z\-]+$).+/;
+const regex =
+	/^(?!generic\([ a-zA-Z\-]+\)$)(?!var\(\s*--[\w-]+\s*\)$)(?!^[a-zA-Z\-]+$).+/;
```

An item is now left unquoted if it is a bare run of letters and hyphens (e.g. `sans-serif`), `generic(...)`, or `var(` + optional whitespace + `--` + word/hyphen characters + optional whitespace + `)`. Examples added to the docblock and tests:

- `var(--my-font), sans-serif` → `var(--my-font), sans-serif`
- `Open Sans, var(--my-font)` → `"Open Sans", var(--my-font)`
- `var( --my-font )` → unchanged (whitespace allowed)
- `var(myfont)` → `"var(myfont)"` (must start with `--`)

The docblock examples were also corrected to match actual output (e.g. `"Mine's"` rather than `"mine's"`, consistent spacing after commas). A new `formatFontFamily` describe block in `test/preview-styles.js` covers these cases plus existing name and keyword behavior. A CHANGELOG entry was added to `packages/global-styles-ui`. The bundle size increase is small (+10 B in `edit-site`, +27 B in `editor`).

## Contribution

The PR is by @ramonjd and may supersede #69926; @nextend started the work and is credited via a co-author trailer. Discussion consists of bot output only (props list, size report, and an unrelated flaky e2e test in `single-file-placeholder-drop.spec.js`). The `var(--name, fallback)` case was deliberately deferred as a TODO because it needs more involved string parsing.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
