# #80166: Icon block: When the default icon is unregistered, nothing is displayed

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @rohitmathur-7
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Icon`
- **Merged:** [`3399e88`](https://github.com/WordPress/gutenberg/commit/3399e88eb5565c097c455ddd21c0ea53abda0968)
- **Discussion:** [#80166](https://github.com/WordPress/gutenberg/pull/80166) · 11 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Icon block now renders a placeholder when its configured icon has been unregistered, instead of displaying a blank space. Previously, the block checked the raw `icon` attribute to decide whether to render, which failed when `wp_unregister_icon()` removed the underlying SVG. This fix ensures the block always provides visual feedback in the editor and on the front end when an icon is missing.

## Impact

- Plugin & theme developers: No action required. If you unregister core icons via `wp_unregister_icon()`, the Icon block will now show a gray placeholder instead of disappearing.
- Site owners & content editors: No action required. Blocks using unregistered icons will display a placeholder rather than a blank area.
- No breaking changes, deprecations, or migration steps are introduced.

## Technical details

The change modifies the render condition in `packages/block-library/src/icon/edit.js`. The original code checked `{ icon ? (` to determine whether to render the `<HtmlRenderer>` component. The diff replaces this with `{ iconToDisplay ? (`, ensuring the block only attempts to render when a resolved SVG string is available. When `iconToDisplay` is falsy (e.g., after `wp_unregister_icon( 'core/info' )`), the block falls through to its default placeholder markup.

```jsx
// Before
{ icon ? (
    <HtmlRenderer html={ iconToDisplay } ... />
) : (
    <Placeholder />
)}

// After
{ iconToDisplay ? (
    <HtmlRenderer html={ iconToDisplay } ... />
) : (
    <Placeholder />
)}
```

## Contribution

Opened by @rohitmathur-7 to address issue #79669. The initial implementation displayed an in-canvas warning when an icon was unregistered. Reviewers (@Mamaduka, @jasmussen, @t-hamano) noted that notices do not scale well in small icon areas and suggested a gray placeholder instead. The author updated the PR to check `iconToDisplay` and render a placeholder, which was merged with co-authors @t-hamano, @Mamaduka, and @jasmussen.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
