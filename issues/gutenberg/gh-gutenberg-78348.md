# #78348: Polish dashboard drag preview motion, elevation, and drop exit.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Experimental`, `[Feature] Dashboard`, `[Package] Grid`
- **Merged:** [`cc21c5d`](https://github.com/WordPress/gutenberg/commit/cc21c5ddef0ea01e5df0c3956a56219513484813)
- **Discussion:** [#78348](https://github.com/WordPress/gutenberg/pull/78348) · 6 comments · 0 reactions

## Summary

The drag-preview overlay in `@wordpress/grid` (`DashboardGrid` and `DashboardLanes`) now uses WPDS motion and elevation tokens for a graduated pick-up animation—shadow scales from `sm` to `lg` on grab—and a smooth drop exit: instead of snapping to its resting state when React tears down the `DragOverlay`, the frame eases `transform` and `box-shadow` back through a new shared `dashboardDragDropAnimation` helper. `prefers-reduced-motion` is respected, applying a static `lg` shadow without scale motion. The change is tagged `[Type] Experimental` and is part of the in-progress Dashboard/Grid feature work.

## Impact

**Plugin & theme developers building on `@wordpress/grid`:**
- `DragOverlay` must be passed `dropAnimation={ dashboardDragDropAnimation }` (exported from `@wordpress/grid`) to get the polished exit; omitting it reverts to `@dnd-kit/core`'s default snap behaviour.
- The drag-preview root element now carries the data attribute `data-wp-dashboard-drag-preview-frame`. This is the supported hook for consumer styling overrides (e.g., scoped `border-radius`).
- `@wordpress/grid` itself does **not** set `border-radius` on the frame; the package is intentionally shape-agnostic. Rounded shadows in the widget dashboard are applied locally and must be replicated by other consumers if desired.
- These APIs are **experimental**—no stability guarantee.

**All other developers / site owners:** No action required.

## Technical details

**New file:** `packages/grid/src/shared/drag-overlay-drop-animation.ts` exports `dashboardDragDropAnimation`. It spreads `defaultDropAnimation` from `@dnd-kit/core`, overrides `duration` and `easing` to match the WPDS `md` / balanced motion token, wraps `defaultDropAnimationSideEffects` to preserve active-element opacity during the drop, and toggles the CSS class `dragPreviewFrameExiting` on the first element matching `[data-wp-dashboard-drag-preview-frame]` for the animation window.

Both `DashboardGrid` and `DashboardLanes` wire it up:

```tsx
// Before
<DragOverlay>
  { /* preview */ }
</DragOverlay>

// After
import { dashboardDragDropAnimation } from './shared/drag-overlay-drop-animation';

<DragOverlay dropAnimation={ dashboardDragDropAnimation }>
  { /* preview */ }
</DragOverlay>
```

**CSS changes in `@wordpress/grid`:**
- **Enter:** `@starting-style` sets `transform: scale(1)` / `box-shadow: var(--wpds-shadow-sm)` so the browser interpolates up to the dragging state (`scale(~1.02)` / `var(--wpds-shadow-lg)`) via a `transition` on `transform` and `box-shadow`.
- **Exit:** `.dragPreviewFrameExiting` resets both properties with `transition-duration` and `transition-timing-function` matching the WPDS `md` / balanced token, replacing the library's default instant teardown.
- **Reduced motion:** `@media (prefers-reduced-motion: reduce)` disables `transform` transitions entirely; `box-shadow` is held statically at `var(--wpds-shadow-lg)` while dragging.

**Widget dashboard (`widgets.module.css`):**
- Tile `box-shadow` progression: resting → `sm`; hover/focus → `md`; grid-resizing (hover suppressed) → `sm`.
- Drag clone tile gets `box-shadow: none`—elevation is carried by the grid frame, preventing shadow stacking on the clone.
- Rounded drag shadow scoped to `.grid :global([data-wp-dashboard-drag-preview-frame])` using `--wpds-border-radius-lg`, aligned with `.tile` / Card radius.

## Contribution

PR #78348 was opened and authored by @jameskoster. @simison performed a rebase and replaced the original screen recordings with updated before/after videos. The six-comment thread consists of the automated size-change report (net 0 B), a flaky-test notice unrelated to this change, and @simison's rebase note. No design alternatives or substantive review debate appear in the discussion. The PR merged at commit `cc21c5d`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
