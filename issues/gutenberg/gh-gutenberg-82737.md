# #82737: UI: Remove automatic Notice announcements

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Breaking Change`, `[Package] Editor`, `[Package] Edit Widgets`, `[Package] UI`, `[Package] Widget Dashboard`
- **Merged:** [`5d46e87`](https://github.com/WordPress/gutenberg/commit/5d46e8770862a6eb54a1608a2e3ccf47fca2fd2e)
- **Discussion:** [#82737](https://github.com/WordPress/gutenberg/pull/82737) · 7 comments · 0 reactions
- **Usefulness:** 5/5

## Summary

The `@wordpress/ui` `Notice.Root` component no longer automatically announces its content to screen readers. The `spokenMessage` and `politeness` props are removed, along with the internal `useSpokenMessage` hook that called `speak()` on mount. Applications that need to announce dynamic updates must now call `speak()` from `@wordpress/a11y` explicitly, choosing the text, timing, and politeness level themselves. Error boundaries in the editor, edit-widgets, and customize-widgets packages were updated to call `speak()` assertively in `componentDidCatch`.

## Impact

- **Plugin & theme developers using `@wordpress/ui` `Notice`:** If you relied on the `spokenMessage` or `politeness` props to trigger screen-reader announcements, those props no longer exist. You must call `speak()` from `@wordpress/a11y` yourself when a dynamic update needs to be announced. The legacy `@wordpress/components` `Notice` is unchanged.
- **Screen-reader users of the block editor, widgets editor, and customize-widgets:** Editor crash announcements now fire via an explicit `speak()` call in the error boundary rather than through the Notice component's built-in mechanism. The announced text excludes action-button labels (e.g. "Try again", "Copy error").
- **No action required** if you only render static `Notice` components without passing `spokenMessage` or `politeness` — the visible output is identical.

## Technical details

In `packages/ui/src/notice/root.tsx`, the diff removes the `speak` import from `@wordpress/a11y`, the `renderToString` and `useEffect` imports from `@wordpress/element`, and three internal helpers: `getDefaultPoliteness()`, `safeRenderToString()`, and the `useSpokenMessage()` hook. The `Root` component's destructured props shrink from `{ intent, children, icon, spokenMessage, politeness, render, ...restProps }` to `{ intent, children, icon, render, ...restProps }`.

Before (removed behavior):
```jsx
// Notice.Root internally called:
useSpokenMessage( spokenMessage, politeness );
// which resolved to:
speak( safeRenderToString( spokenMessage ), politeness );
```

After: `Notice.Root` renders purely visual markup with no live-region roles and no `speak()` call. The JSDoc now states: "It does not announce its content to assistive technology. Consumers are responsible for announcing dynamic updates, for example with `speak()` from `@wordpress/a11y`."

In the three error-boundary files (`packages/editor/src/components/error-boundary/index.jsx`, `packages/edit-widgets/src/components/error-boundary/index.jsx`, `packages/customize-widgets/src/components/error-boundary/index.jsx`), a new `getErrorNotice()` helper returns the title and description strings, and `componentDidCatch` now calls:

```js
const { title, description } = getErrorNotice();
speak( `${ title }. ${ description }`, 'assertive' );
```

`@wordpress/a11y` is added as a direct dependency in `customize-widgets`, `edit-widgets`, and `editor` `package.json` files. A new Storybook page at `packages/ui/src/notice/stories/announcements.mdx` documents the migration path with static, `role="alert"`, and `role="status"` examples.

## Contribution

Opened by @ciampo referencing issue #82701. @simison suggested either adding Storybook documentation with `role` examples or keeping the `spokenMessage`/`politeness` API even while removing the automatic announcement. @ciampo responded by adding the Storybook migration guidance. @mirka proposed basing the recommended pattern on `speak()` rather than ARIA live-region roles, and @ciampo agreed, updating the error boundaries and examples accordingly. @joedolson reinforced the single-`speak()`-route approach and noted that `role="alert"` semantics are still under active discussion in the W3C ARIA spec (w3c/aria#2153). @aduth flagged a changelog-validation failure from a concurrent PR (#83043) that required a rebase. Merged as `5d46e87`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
