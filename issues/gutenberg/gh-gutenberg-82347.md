# #82347: UI: Support custom targets on Link and Menu.LinkItem

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`59876b1`](https://github.com/WordPress/gutenberg/commit/59876b1c6cb5e06db84b24eb3c167c0b2d4c33d0)
- **Discussion:** [#82347](https://github.com/WordPress/gutenberg/pull/82347) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/ui` `Link` and `Menu.LinkItem` components now accept a native `target` prop. An explicit `target` decides the browsing context, so named targets such as `wp-preview-123` pass through unchanged, where previously `openInNewTab` always overwrote the target with `_blank`. `target="_blank"` now gets the same external-link indicator and "(opens in a new tab)" accessible notice as `openInNewTab`. `openInNewTab` remains the preferred way to open a new tab.

## Impact

**Plugin & theme developers using `@wordpress/ui`**
- Additive change; no deprecations or removals. Existing `openInNewTab` usage behaves identically.
- You can now reuse a named tab or window (e.g. a preview window) without custom link rendering.
- Behavior to note: if both `target` and `openInNewTab` are set, `target` wins, and the new-tab indicator and notice still render because `openInNewTab` is true. A non-`_blank` named target combined with `openInNewTab` therefore shows the "opens in a new tab" notice for a link that opens in a named context.
- Passing `target="_blank"` alone now shows the indicator and notice, which it did not before (the `target` prop was previously omitted from the typed props).

**Site owners / hosting / REST consumers**
- No action required.

## Technical details

Changes are in `packages/ui/src/link/link.tsx` and `packages/ui/src/menu/link-item.tsx`, with prop types in `link/types.ts` and `menu/types.ts`.

Both components destructure `target` and compute:

```tsx
const shouldShowNewTabIndicator = openInNewTab || target === '_blank';
// rendered attribute
target: target ?? ( openInNewTab ? '_blank' : undefined )
```

Before, the attribute was `openInNewTab ? '_blank' : undefined` and the indicator depended solely on `openInNewTab`.

Types: `LinkProps` already used `Omit< ComponentProps< 'a' >, 'target' >`; it now re-adds `target?: ComponentProps< 'a' >[ 'target' ]` with documented precedence. `LinkItemProps` gets the same prop. The `openInNewTab` docs are reworded to describe the indicator/notice and the `_blank` default.

Also changed: jsdom tests for `_blank`, named targets (no notice), and explicit-target precedence in both components; a new `OpenInNewTab` Link story; the Menu `LinkItem` story now sets a named target alongside `openInNewTab`; `target` added to `storybook/components-manifest.yml`; and a `packages/ui/CHANGELOG.md` entry.

## Contribution

The PR is a follow-up to #82321 and was implemented with AI assistance in Codex and reviewed locally by the author. The discussion on the PR consists only of bot comments (CodeRabbit, the bundle-size/performance report, and props-bot), with no recorded design debate. Props-bot lists @mirka, @Mamaduka, and @jsnajdr as interacting with the PR.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
