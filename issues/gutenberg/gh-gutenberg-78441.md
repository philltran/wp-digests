# #78441: @wordpress/ui: Compat overlay slot — viewport-sized containing block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] UI`
- **Merged:** [`61ace80`](https://github.com/WordPress/gutenberg/commit/61ace80f93e5b15792b75407da689ab86d1f5a0d)
- **Discussion:** [#78441](https://github.com/WordPress/gutenberg/pull/78441) · 3 comments · 0 reactions

## Summary

Fixes a layout collapse in the `@wordpress/ui` compat overlay slot that caused portaled popups to shrink to their min-content width, resulting in long text wrapping word-by-word. By establishing a proper viewport-sized containing block and correctly layering pointer events, the slot now provides adequate space for floating UI while preventing click-through interference. This resolves visual regressions surfaced during migration to the new `@wordpress/ui` Tooltip.

## Impact

- **Plugin & Theme Developers (UI/Editor):** Components that route their portals into the compat overlay (`Tooltip`, `Select`, `Autocomplete`) will now render with correct layout sizing inside modals/popovers.
- **No breaking changes:** The adjustment is purely CSS-driven and transparent for standard component usage. No API or configuration changes are required.
- **Custom override caution:** Developers manually overriding the compat slot's container styles should verify that the new `pointer-events: none` on the parent and `pointer-events: auto` on `.slot > *` do not conflict with existing CSS cascade strategies or custom `@layer` scoping.

## Technical details

- **File modified:** `packages/ui/src/utils/css/wp-compat-overlay-slot.module.css`
- **Root cause:** The slot previously used `position: fixed; top: 0; left: 0;` without explicit dimensions. Since its only child uses `position: absolute`, it contributed nothing to the parent's intrinsic size, collapsing the container to `0×0`. Absolutely positioned children inside a zero-width block fall back to shrink-to-fit against zero available width, causing min-content wrapping.
- **Behavioral shift:** The slot is now `position: fixed; inset: 0; pointer-events: none;`, establishing a full-bleed viewport-sized containing block. A child selector rule `.slot > * { pointer-events: auto; }` is applied within the `@layer wp-ui-utilities` scope to re-enable interaction for portaled overlays while keeping the parent layer non-interfering.
- **Scope:** Only affects components explicitly opting into the compat slot (`Tooltip`, `Autocomplete`, `Select`). Base UI components (`Popover`, `Dialog`, `AlertDialog`, `Drawer`) that portal directly to `document.body` remain unaffected by this CSS change.

## Contribution

Opened and merged by @ciampo to address visual regressions in portaled popups that emerged during the `@wordpress/ui` Tooltip migration (#78411). The PR includes validation against existing Storybook debug fixtures and local unit tests, with explicit confirmation that keyboard interaction surfaces remain unchanged. Merged at `61ace80`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
