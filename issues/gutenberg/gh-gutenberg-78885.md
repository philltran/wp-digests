# #78885: Popover: add open/close motion and fix close re-anchor

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `Design System`, `[Package] UI`
- **Merged:** [`c34d60a`](https://github.com/WordPress/gutenberg/commit/c34d60a7614eb8d3c36822503f4c6aee783c5810)
- **Discussion:** [#78885](https://github.com/WordPress/gutenberg/pull/78885) · 17 comments · 1 reactions

## Summary

Adds open/close animation to the `@wordpress/ui` `Popover` component. The update applies the shared `dropdown-motion` slide/fade profile to `Popover.Popup`, adds a backdrop opacity fade, and splits the popup DOM into a transformed motion layer (`.popup`) and an inner surface div (`.surface`) to prevent arrow alignment drift during transforms. While the PR title references a close-reposition fix, that change was explicitly excluded during review to address the root cause separately.

## Impact

- **Theme & Plugin Developers**: Default non-`unstyled` `Popover` instances now animate open/close. Animation respects the system's `prefers-reduced-motion` setting.
- **Developers using `variant="unstyled"`**: No behavioral or structural changes; this variant intentionally skips the motion wrapper and remains unchanged.
- **Custom Implementations**: The DOM structure of `Popover.Popup` changed to include a `.surface` wrapper around children. Developers relying on direct child selectors or measuring popup geometry directly may need to adjust selectors to target `.surface`.
- No migration code required for standard usage; animation is opt-out via CSS tokens or OS settings.

## Technical details

- `packages/ui/src/popover/popup.tsx`: Introduces a `useDefaultSurface` flag (`variant !== 'unstyled'`). When true, children are wrapped in `<div className={ styles.surface }>` and `PopoverValidationProvider` is moved inside the surface. The motion class is applied conditionally: `useDefaultSurface && styles.popup`.
- `packages/ui/src/popover/style.module.css`: 
  - `.popup` now composes `dropdown-motion from "../utils/css/dropdown-motion.module.css"` and carries the CSS transform layer.
  - Visual chrome (background, border-radius, padding, shadow, outline) moved to `.surface` to ensure it does not establish a containing block for `Popover.Arrow`.
  - Selector updated from `.backdrop ~ * .popup` to `.backdrop ~ * .surface` to toggle border visibility when the backdrop is present.
- Backdrop fade logic uses `data-starting-style` and `data-ending-style` attribute guards (`&[data-open]:not([data-starting-style]):not([data-ending-style]) { opacity: 1; }`) paired with a `prefers-reduced-motion` media query to prevent snap-in on mount.
- The split architecture prevents the arrow from resolving against the padding box of a transformed element, which would cause it to sit ~1px inside the border during animation.

## Contribution

Opened by @jameskoster; merged after review feedback from @ciampo, @fcoveram, and @aduth. Design discussions centered on whether a low-level primitive like `Popover` should ship with a default animation, given that higher-level overlays (menus, tooltips, dialogs) typically require context-aware motion. The team proceeded with a default slide/fade while noting that bespoke animations may still be needed per use case. An early proposal to fix a close-time repositioning bug was explicitly reverted during review to avoid masking the underlying cause; that fix is deferred to a separate PR.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
