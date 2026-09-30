# #83347: Components: Deprecate ResponsiveWrapper

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Tool] ESLint plugin`
- **Merged:** [`7622f50`](https://github.com/WordPress/gutenberg/commit/7622f502214b67a36ace70622039a2784d88932a)
- **Discussion:** [#83347](https://github.com/WordPress/gutenberg/pull/83347) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `ResponsiveWrapper` component in `@wordpress/components` is formally deprecated with a runtime console warning, planned for removal in WordPress 7.4. The recommended replacement is the native CSS `aspect-ratio` property. The component was already marked "not recommended" in Storybook, and this change adds the `deprecated()` call so external consumers receive an explicit warning before the removal.

## Impact

- **Plugin & theme developers using `ResponsiveWrapper` from `@wordpress/components`:** A console deprecation warning will fire on every render starting in 7.2. The component will be removed in 7.4. Replace usage with the CSS `aspect-ratio` property on the wrapping element.
- **Projects using the Gutenberg ESLint config:** The `use-recommended-components` rule now reports `Use the CSS aspect-ratio property instead.` for any import of `ResponsiveWrapper`, replacing the previous vague "planned for deprecation" message.
- **No action required** for developers who do not import `ResponsiveWrapper` from `@wordpress/components`. The navigation block's internal local wrapper of the same name is unaffected.

## Technical details

In `packages/components/src/responsive-wrapper/index.tsx`, a call to `deprecated()` from `@wordpress/deprecated` is inserted at the top of the `ResponsiveWrapper` render function:

```tsx
deprecated( 'wp.components.ResponsiveWrapper', {
	since: '7.2',
	version: '7.4',
	alternative: 'the CSS aspect-ratio property',
} );
```

The JSDoc block gains an `@deprecated` tag. The Storybook entry in `stories/index.story.tsx` moves from the `Layout` category to `Deprecated` and updates its `componentStatus.notes` to `Deprecated. Use the CSS \`aspect-ratio\` property instead.`

In `packages/eslint-plugin/rules/use-recommended-components.js`, the `DENYLIST` entry for `ResponsiveWrapper` changes from:

```js
ResponsiveWrapper: '{{ name }} is planned for deprecation.',
```
to:
```js
ResponsiveWrapper: 'Use the CSS `aspect-ratio` property instead.',
```

The README in `packages/components/src/responsive-wrapper/README.md` gains a deprecation callout at the top. Bundle size impact is +46 B in `build/scripts/components/index.min.js`.

## Contribution

Opened by @mirka with co-authorship from @jorgefilipecosta. The PR carries only 2 comments and 0 reactions, indicating a straightforward, uncontested change. It is part of a coordinated batch of component deprecations targeting WordPress 7.4 (the same changelog section lists `Scrollable`, `Elevation`, and `Divider` deprecations in adjacent PRs). No alternative approach or design debate is visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
