# #79849: Global Styles: Show skeleton placeholder while previews load

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @juanfra
- **Labels:** `[Type] Enhancement`, `Global Styles`
- **Merged:** [`e0f831a`](https://github.com/WordPress/gutenberg/commit/e0f831a89c80c041621bf1bfb6748b966d9d153f)
- **Discussion:** [#79849](https://github.com/WordPress/gutenberg/pull/79849) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Global Styles UI now renders a `Skeleton` placeholder in the `PreviewWrapper` component while the preview container's dimensions are being calculated. Previously, the preview area remained heightless until a resize observer measured the container, causing a noticeable layout jump when the actual preview rendered. This change reserves the correct height immediately, providing a smoother user experience in the Site Editor and block editor style panels.

## Impact

- **Site owners & editors:** No action required. The change improves the visual stability of the Global Styles preview panels in the Site Editor and block editor.
- **Plugin & theme developers:** No action required. This is an internal UI enhancement to the `@wordpress/global-styles-ui` package and does not expose new public APIs or hooks.
- **Hosting & platform teams:** No action required. No configuration or migration steps are needed.

## Technical details

The change modifies `packages/global-styles-ui/src/preview-wrapper.tsx` to conditionally render a `Skeleton` component from `@wordpress/ui` when `! isReady`. The skeleton is assigned the `global-styles-ui-preview__wrapper` class and an inline `aspectRatio` style calculated from `normalizedWidth / normalizedHeight`, matching the eventual preview dimensions. This prevents the DOM from collapsing to zero height during the resize observer measurement phase.

```tsx
// Before
<div style={ { position: 'relative' } }>
    { containerResizeListener }
</div>
{ isReady && (
    <div className={ clsx( 'global-styles-ui-preview__wrapper', { ... } }>
        {/* preview content */}
    </div>
)}

// After
<div style={ { position: 'relative' } }>
    { containerResizeListener }
</div>
{ ! isReady && (
    <Skeleton
        className="global-styles-ui-preview__wrapper"
        style={ {
            aspectRatio: normalizedWidth / normalizedHeight,
        } }
    />
)}
{ isReady && (
    <div className={ clsx( 'global-styles-ui-preview__wrapper', { ... } }>
        {/* preview content */}
    </div>
)}
```

No new hooks, filters, or REST endpoints are introduced. The change is isolated to the internal preview rendering logic.

## Contribution

Opened by @juanfra to address layout jumps reported in issue #79284. The implementation leverages the recently introduced `Skeleton` component from `@wordpress/ui` (PR #79671) to reserve space during the resize observer measurement phase. The PR was reviewed and manually merged by @ciampo after GitHub's auto-merge failed, with co-authors @t-hamano, @mirka, and @SainathPoojary. No alternative approaches were debated in the provided discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
