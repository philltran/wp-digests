# #83423: Components: Stop rendering a hidden drag preview and a media query per draggable

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Components`
- **Merged:** [`5b04c1b`](https://github.com/WordPress/gutenberg/commit/5b04c1b7ebdc4a9a2eae80673e0ea81d4372c3ee)
- **Discussion:** [#83423](https://github.com/WordPress/gutenberg/pull/83423) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `Draggable` component in `@wordpress/components` no longer renders a hidden `display: none` div containing the `__experimentalDragComponent` for every instance. Instead, it portals the drag component into a clone wrapper element only while a drag is in progress, eliminating hundreds of unnecessary DOM nodes in the block inserter. Separately, `Flex` and `Grid` no longer call `window.matchMedia()` and subscribe a `resize` listener when their props are scalar values rather than breakpoint arrays. A related bug fix ensures `cleanupRef.current` resets to a no-op after running, preventing a stale cleanup from stripping the `is-dragging-components-draggable` body class from a concurrent drag.

## Impact

- **Site owners / editors:** No action required. The block inserter will open with less main-thread work (fewer passive effects, fewer media query subscriptions), reducing post-open jank and keystroke latency.
- **Plugin & theme developers:** No API or prop changes. `__experimentalDragComponent` still accepts the same React element and renders identically during a drag. One behavioral note: a `dragComponent` containing `<img>` elements will now load at drag start rather than immediately on mount (the stock `BlockDraggableChip` uses inline SVG, so this is a non-issue for core).
- **Hosting & platform:** No configuration or migration needed. Bundle size change is +6 B net.

## Technical details

**`packages/components/src/draggable/index.tsx`**

The hidden preview div and its ref are removed:

```tsx
// Before
const dragComponentRef = useRef<HTMLDivElement>(null);
// …
{ dragComponent && (
  <div className="components-draggable-drag-component-root"
       style={{ display: 'none' }}
       ref={ dragComponentRef }>
    { dragComponent }
  </div>
)}
```

```tsx
// After
const [ dragComponentContainer, setDragComponentContainer ] =
  useState<HTMLDivElement | null>(null);
// …
{ dragComponentContainer &&
  createPortal( dragComponent, dragComponentContainer ) }
```

Inside `start()`, the old code cloned `dragComponentRef.current.innerHTML` into a new `div` and appended it to `cloneWrapper`. The new code simply calls `setDragComponentContainer(cloneWrapper)`; the portal renders the live React tree into that element. Because `dragstart` is a discrete React event, the state update commits synchronously before the handler returns. `cleanupRef.current` now ends with `cleanupRef.current = () => {}` so an unmount after a completed drag cannot re-run the stale cleanup.

**`packages/components/src/utils/use-responsive-value.ts`**

`useBreakpointIndex` gains an `enabled` option (default `true`). When `false`, the effect body returns early—no `matchMedia`, no `resize` listener, no `setValue` loop. `useResponsiveValue` computes:

```ts
const isResponsive = Array.isArray( values ) && values.length > 1;
const index = useBreakpointIndex( { ...options, enabled: isResponsive } );
```

so scalar props (the common case in `Flex`/`Grid`) skip all viewport watching.

## Contribution

Opened by @Mamaduka while debugging #73121; a performance probe surfaced the per-draggable hidden preview and per-instance media query subscriptions. @andrewserong reviewed. The PR notes AI assistance (Claude). Merged as `5b04c1b` with no notable design debate in the three recorded comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
