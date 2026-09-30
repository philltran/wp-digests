# #82379: Components: deprecate Badge

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @simison
- **Labels:** `[Type] Enhancement`, `[Feature] UI Components`, `[Package] Components`, `[Tool] ESLint plugin`, `Storybook`
- **Merged:** [`ab4c8d4`](https://github.com/WordPress/gutenberg/commit/ab4c8d4210ba44b2ceb7c4bc4cb21ed42133e760)
- **Discussion:** [#82379](https://github.com/WordPress/gutenberg/pull/82379) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The private `Badge` component in `@wordpress/components` is now formally deprecated, with a runtime `deprecated()` warning directing consumers to `Badge` from `@wordpress/ui`. The deprecation follows the same pattern established for `ButtonGroup`, `ValidatedInputControl`, and `Surface`: a console warning, Storybook status change, README callout, and an ESLint denylist entry. All internal Gutenberg usages were migrated in a prerequisite issue (#82440) before the warning was enabled, so the console log will not break existing Gutenberg tests.

## Impact

- **Gutenberg contributors / plugin developers using private APIs:** Importing `Badge` from `@wordpress/components` private APIs now emits a `deprecated()` console warning (`wp.components.privateApis.Badge`, since `7.2`). Replace with `Badge` from `@wordpress/ui`. The component is slated for removal "within a few Gutenberg plugin releases."
- **ESLint users of `@wordpress/eslint-plugin`:** The `use-recommended-components` rule now flags `Badge` imports from `@wordpress/components` and suggests the `@wordpress/ui` equivalent.
- **Site owners / theme developers:** No action required. `Badge` was a private (locked) API and was not intended for external use.
- **No breaking change yet:** The component still renders; only the warning and documentation surfaces changed.

## Technical details

In `packages/components/src/badge/index.tsx`, the `Badge` function now calls:

```ts
import deprecated from '@wordpress/deprecated';

// inside Badge():
deprecated( 'wp.components.privateApis.Badge', {
	since: '7.2',
	alternative: 'Badge from @wordpress/ui',
	hint: 'This private API will be completely removed within a few Gutenberg plugin releases.',
} );
```

Supporting changes:

- **Docs schema** (`packages/components/schemas/docs-manifest.json`): new optional `deprecated` string property. When present, `tools/docs/gen-components-docs/markdown/index.mjs` renders a `<p class="callout callout-alert">` block in the generated README.
- **Badge manifest** (`packages/components/src/badge/docs-manifest.json`): sets `"deprecated": "Please use `Badge` from the `@wordpress/ui` package instead."`.
- **Storybook** (`packages/components/src/badge/stories/index.story.tsx`): title moved from `Components/@wordpress-components/Containers/Badge` to `Components/@wordpress-components/Deprecated/Badge`; `componentStatus.status` changed from `'use-with-caution'` to `'not-recommended'`.
- **ESLint** (`packages/eslint-plugin/rules/use-recommended-components.js`): `Badge` added to the `DENYLIST` object with the message `'Use `{{ name }}` from `@wordpress/ui` instead.'`.
- **Test** (`packages/components/src/badge/test/index.jsdom.test.tsx`): new `describe('Shows a deprecation warning')` block that clears the `logged` map, renders `<Badge>`, and asserts `console` received the exact deprecation string. Existing render tests are suppressed via `beforeEach`/`afterEach` on `logged[DEPRECATION_MESSAGE]`.

## Contribution

Opened by @simison. @mirka flagged in review that the console warning should not ship until all internal Gutenberg usages were migrated, because the warning would break existing jsdom tests. @simison tracked that migration in issue #82440 and confirmed completion before requesting a second look. The deprecation test initially broke after a rebase and was rewritten to match the existing `ValidatedInputControl` and `Surface` test patterns. Co-authored with @ciampo and @mirka.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
