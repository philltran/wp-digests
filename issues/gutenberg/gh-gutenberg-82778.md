# #82778: Fix Stylelint config and coding standards for font weight numeric values and disallow relative weights

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @afercia
- **Labels:** `[Type] Bug`, `[Package] Components`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`, `[Tool] stylelint config`, `[Package] Base styles`, `[Package] DataViews`
- **Merged:** [`6caf41c`](https://github.com/WordPress/gutenberg/commit/6caf41cad16582429f1f6d5e7ca96065e2f89ec3)
- **Discussion:** [#82778](https://github.com/WordPress/gutenberg/pull/82778) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The shared `@wordpress/stylelint-config` package now sets `font-weight-notation` to plain `'numeric'`, dropping the previous `ignore: [ 'relative' ]` option, so `bolder` and `lighter` are flagged along with keywords like `normal` and `bold`. Gutenberg's own internal `tools/stylelint/config.js` had this rule set to `null`. It is now `'numeric'` as well. The PR also converts the existing keyword weights in Gutenberg SCSS (`normal` to `400`, `bold` to `700`) so `npm run lint:css` stays clean.

## Impact

- **Plugin & theme developers using `@wordpress/stylelint-config`:** This is listed under **Breaking Changes** in the package CHANGELOG. After upgrading, `font-weight: bolder` and `font-weight: lighter` produce `font-weight-notation` errors, as do `normal` and `bold`. Replace them with numeric values (e.g. `400`, `700`). Keyword values were already flagged before this change, so the new failures come from relative weights.
- **Gutenberg contributors:** The repo's internal stylelint config now enforces numeric weights, so new SCSS using `normal`/`bold` will fail `npm run lint:css`.
- **Site owners / end users:** No action required. The compiled CSS change is cosmetic and equivalent (`normal` = 400, `bold` = 700), and bundle sizes shift by a few bytes.
- **Hosting & headless:** Not affected.

## Technical details

**Config changes**

`packages/stylelint-config/index.js`:

```js
// before
'font-weight-notation': [ 'numeric', { ignore: [ 'relative' ] } ],
// after
'font-weight-notation': 'numeric',
```

`tools/stylelint/config.js` (Gutenberg's internal config): `'font-weight-notation': null` becomes `'numeric'`.

**Codebase cleanup** to satisfy the rule:
- `font-weight: normal` becomes `400` and `bold` becomes `700` across SCSS in base-styles, block-directory, block-editor, block-library (comments, navigation, search), components (placeholder), dataviews (table), editor, global-styles-ui and widgets.
- The `font` shorthand with `normal` is also converted in `packages/base-styles/_mixins.scss` and `packages/editor/src/components/autocompleters/style.scss` (`font: normal 30px/1 dashicons` becomes `font: 400 30px/1 dashicons`).

**Tests** (`packages/stylelint-config/test/`):
- `bolder` is removed from `values-valid.css` and added to `values-invalid.css`.
- The expected warning count in `values.js` goes from 9 to 10.
- The snapshot in `__snapshots__/values.js.snap` adds the new `font-weight-notation` warning and shifts subsequent line numbers.

No hooks, REST schema, or PHP are involved.

## Contribution

The PR closes issue #82777. During review, @mirka asked that `bolder` be added specifically to the invalid-values test, since it was the relative value that had previously been allowed. The existing invalid fixture only covered `bold`. @afercia noted the test suite already expected numeric-only notation, so the real inconsistency was the internal config setting the rule to `null`. He added `bolder` to the invalid fixture and moved the changelog entry under Breaking Changes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
