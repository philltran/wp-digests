# #83344: Components: Deprecate ZStack

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Tool] ESLint plugin`
- **Merged:** [`eb60ef9`](https://github.com/WordPress/gutenberg/commit/eb60ef9a51f03bac99d546c6e81ed7d4d3740dfb)
- **Discussion:** [#83344](https://github.com/WordPress/gutenberg/pull/83344) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `__experimentalZStack` component in `@wordpress/components` is formally deprecated with a runtime `deprecated()` warning (since 7.2, removal planned for 7.4). The component was already marked not-recommended in Storybook and flagged by the `use-recommended-components` ESLint rule; this PR makes the deprecation explicit at render time and moves the Storybook entry under the Deprecated group. No replacement component exists in `@wordpress/ui` — the stated alternative is writing your own CSS.

## Impact

- **Plugin & theme developers using `__experimentalZStack`:** A console warning will now fire on every render: `wp.components.__experimentalZStack is deprecated since version 7.2 and will be removed in version 7.4. Please use your own CSS instead.` Replace usage with custom CSS (e.g. `position: relative` / `z-index` on child elements). The component will be removed in WordPress 7.4.
- **ESLint users of `@wordpress/eslint-plugin`:** The `use-recommended-components` rule message for `__experimentalZStack` changes from `'{{ name }} is planned for deprecation. Write your own CSS instead.'` to `'Write your own CSS instead.'` — the rule still fires, only the wording tightens.
- **Storybook consumers:** The story moves from `Components/@wordpress-components/ZStack` to `Components/@wordpress-components/Deprecated/ZStack`. The story `id` remains `components-zstack`, so existing permalinks (`?path=/story/components-zstack--default`) continue to work.
- **No action required** if you are not importing `__experimentalZStack`.

## Technical details

In `packages/components/src/z-stack/component.tsx`, a `deprecated()` call from `@wordpress/deprecated` is inserted at the top of `UnconnectedZStack`:

```tsx
deprecated( 'wp.components.__experimentalZStack', {
	since: '7.2',
	version: '7.4',
	alternative: 'your own CSS',
} );
```

The JSDoc block gains a `@deprecated` tag and a prose note. The Storybook meta in `packages/components/src/z-stack/stories/index.story.tsx` changes `title` to `'Components/@wordpress-components/Deprecated/ZStack'` and updates `componentStatus.notes` from `'Planned for deprecation. Write your own CSS instead.'` to `'Deprecated. Write your own CSS instead.'`. The `id` field is unchanged (`'components-zstack'`).

In `packages/eslint-plugin/rules/use-recommended-components.js`, the `DENYLIST` entry for `__experimentalZStack` is simplified from `'{{ name }} is planned for deprecation. Write your own CSS instead.'` to `'Write your own CSS instead.'`, and the corresponding test expectations in `use-recommended-components.js` are updated to match. Bundle size impact is +15 B in `build/scripts/components/index.min.js`.

## Contribution

Opened by @mirka and co-authored with @Mamaduka. The PR carries no substantive design debate in its two comments; it is a straightforward deprecation following the same pattern applied to `ResponsiveWrapper`, `Scrollable`, and `Elevation` in the same release cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
