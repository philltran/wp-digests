# #77956: Modal: Render as a bottom sheet on mobile

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @crisbusquets
- **Labels:** `[Type] Enhancement`, `[Package] Components`
- **Merged:** [`e02fee9`](https://github.com/WordPress/gutenberg/commit/e02fee931d9017dcae6b7c1e0278d7627d97809d)
- **Discussion:** [#77956](https://github.com/WordPress/gutenberg/pull/77956) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `Modal` component in `@wordpress/components` now renders as a bottom sheet on mobile viewports (<600px), anchored to the bottom of the viewport with edge-to-edge width and rounded top corners only. This layout shift improves mobile ergonomics by keeping primary CTAs within the thumb zone and preventing short-content modals from stretching to fill the entire screen. Desktop rendering, sizing, and animations remain completely unchanged.

## Impact

- **Plugin & theme developers**: No public API changes or breaking behavior. Existing `<Modal>` implementations automatically adopt the new layout without code updates.
- **Site owners / end users**: Improved touch-target positioning and reduced empty space in admin UI modals on mobile devices.
- **Hosting & platform teams**: No action required.

## Technical details

Changes are isolated to `packages/components/src/modal/style.scss` and modify the `.components-modal__frame` layout below `break-small` (600px):

```scss
// Before (mobile)
.components-modal__frame { margin: $grid-unit-50 0 0 0; }

// After (mobile)
.components-modal__frame { 
  align-self: flex-end; 
  margin: 0; 
  width: 100%; 
  max-height: calc(100% - #{$grid-unit-50}); 
  border-radius: $radius-large $radius-large 0 0;
}
```

The `&.is-full-screen` block is consolidated into a single definition with viewport-specific overrides via `@include break-small()` and `@include break-medium()`. Keyframe animations (`components-modal__appear-animation`, `components-modal__disappear_animation`) default to `translateY(100%) ↔ 0` on mobile, falling back to the original `scale(0.9) ↔ 1` inside `@include break-small()`; identical names preserve the `animationend` listener in `use-modal-exit-animation.ts` without JS modifications. Reduced motion remains respected via existing media guards.

## Contribution

Opened by @crisbusquets and refined after review from @ciampo and @t-hamano. The PR was initially flagged for an outdated base commit and stylelint errors, which were resolved via rebasing and consolidating nested `@include break-small()` blocks. During review, maintainers noted UX friction when dynamically updating modal content on mobile; while a per-instance min-height workaround was suggested, the final diff ships only the CSS bottom-sheet implementation without altering JS animation logic or component props.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
