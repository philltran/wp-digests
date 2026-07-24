# #77563: Navigation: Fixes `aria-expanded` not updating on hover submenu inside overlay

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @himanshupathak95
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Package] Block library`, `[Block] Navigation`, `Backport to WP 7.1 Beta/RC`
- **Merged:** [`0a999d7`](https://github.com/WordPress/gutenberg/commit/0a999d7554efd7bec0522f08a46a1d5353b32a14)
- **Discussion:** [#77563](https://github.com/WordPress/gutenberg/pull/77563) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes an accessibility bug in the Navigation block where the submenu toggle button’s `aria-expanded` attribute remained `false` when hovering over a submenu inside an already-open overlay. The change introduces a new `isSubmenuOpen` state getter and removes a guard that previously blocked hover state updates when the overlay was active. This ensures screen readers correctly announce the expanded state of submenus in hover-mode overlays.

## Impact

- **Theme & plugin developers:** No breaking changes or migration steps required. The fix applies automatically to sites using the Navigation block with `Submenu Visibility` set to `Hover` inside an overlay.
- **Accessibility auditors & screen reader users:** Resolves a mismatch between visual submenu expansion and ARIA state, preventing false "closed" announcements in hover-mode overlays.
- **No action required** for existing implementations; the behavior is corrected on update.

## Technical details

The diff modifies three files in `packages/block-library/src/navigation/`:
- `index.php`: Binds the submenu toggle button’s `data-wp-bind--aria-expanded` directive to `state.isSubmenuOpen` instead of `state.isMenuOpen`.
- `view.js`: Adds an `isSubmenuOpen` getter to the Interactivity store state that returns `true` if `ctx.overlayOpenedBy` contains any truthy values or if `state.isMenuOpen` is true. It also removes the `overlayOpenedBy` guard from `openMenuOnHover` and `closeMenuOnHover`, allowing hover state transitions to fire even when the parent overlay is already open.
- `test/view.js`: Introduces a Jest test verifying that `submenuOpenedBy.hover` and `state.isSubmenuOpen` correctly track hover enter/leave events inside an open overlay.

Before/after for the directive binding:
```diff
- $tags->set_attribute( 'data-wp-bind--aria-expanded', 'state.isMenuOpen' );
+ $tags->set_attribute( 'data-wp-bind--aria-expanded', 'state.isSubmenuOpen' );
```

## Contribution

Opened by @himanshupathak95 to address issue #76658. During review, @jeryj questioned whether the attribute should update when the overlay itself is open, prompting the author to add the `isSubmenuOpen` state getter and clarify the overlay’s visual expansion behavior. @getdave requested low-level test coverage, which was added in a subsequent commit. The PR was rebased, approved, and marked for backport to WordPress 7.1 Beta/RC.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
