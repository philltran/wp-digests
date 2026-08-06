# #81083: Link Control: Restore the preview title underline

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Backported to WP Core`
- **Merged:** [`e383d02`](https://github.com/WordPress/gutenberg/commit/e383d02d18c61571b6b665fdb4a1586d620b8a1b)
- **Discussion:** [#81083](https://github.com/WordPress/gutenberg/pull/81083) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Link Control component in the Block Editor now correctly renders the underline on preview titles and link text. A prior styling update added `text-underline-offset` to align with the `ui/Link` component, but the tight `line-height` on the preview title caused the underline to be clipped by the parent container's overflow constraints. This fix widens the line height to ensure the underline remains visible.

## Impact

- **Block editor users**: Preview titles and inline link text in popovers now display with visible underlines, resolving a visual regression.
- **Plugin & theme developers**: No code changes required. The fix is applied internally to the `LinkControl` component.
- **Hosting & platform teams**: No action required.

## Technical details

The change modifies `packages/block-editor/src/components/link-control/style.scss`. It replaces the hardcoded `line-height: 1.1;` on `.block-editor-link-control__preview-title` with the design system token `var(--wpds-typography-line-height-sm)`. This increases the vertical spacing just enough to prevent the underline (introduced via `text-underline-offset: 0.2em` in a previous PR) from being clipped by the `Truncate` component's `overflow: hidden` constraints.

```css
/* Before */
.block-editor-link-control__preview-title {
	line-height: 1.1;
}

/* After */
.block-editor-link-control__preview-title {
	line-height: var(--wpds-typography-line-height-sm);
}
```

## Contribution

Opened by @t-hamano as a follow-up to #77790, the PR was reviewed and merged with a single CSS adjustment. During review, @aduth noted a related descender-clipping issue on the URL path below the title, which @t-hamano acknowledged as a known, OS-specific rendering quirk that falls outside this fix's scope. The change was subsequently cherry-picked to the `wp/7.1` branch for inclusion in the next WordPress release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
