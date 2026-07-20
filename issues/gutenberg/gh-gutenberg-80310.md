# #80310: UI: Keep the compat overlay slot accessible inside Modal

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] Components`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Package] UI`
- **Merged:** [`cd518d9`](https://github.com/WordPress/gutenberg/commit/cd518d91f7a82ed4988317a8a264078073e79287)
- **Discussion:** [#80310](https://github.com/WordPress/gutenberg/pull/80310) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes an accessibility bug where overlays rendered into the `@wordpress/ui` compat overlay slot (`[data-wp-compat-overlay-slot]`) become hidden from assistive technologies when a `@wordpress/components` Modal is open. The Modal sets `aria-hidden="true"` on the slot, but this change explicitly resets the slot to `aria-hidden="false"` so nested overlays remain exposed and operable.

## Impact

- Plugin & theme developers using `@wordpress/ui` alongside `@wordpress/components`: Overlays (such as `Popover`) rendered inside a `Modal` will now correctly remain accessible to screen readers and keyboard navigation.
- No code changes or configuration updates required. The fix is applied internally within the slot utility.
- No breaking changes, deprecations, or migration steps.

## Technical details

The change modifies `packages/ui/src/utils/wp-compat-overlay-slot.ts` by introducing `ensureSlotIsAccessible()`, which forces `aria-hidden="false"` on the compat overlay slot element. This function is now called in all three retrieval paths of `getWpCompatOverlaySlot()`: when returning a cached slot, when adopting an existing DOM element matching `WP_COMPAT_OVERLAY_SLOT_ATTRIBUTE`, and when creating a new slot. Because the `@wordpress/components` Modal preserves the `aria-hidden` state of body children, explicitly resetting it prevents the Modal's accessibility isolation from hiding the slot. Unit tests in `packages/ui/src/utils/test/wp-compat-overlay-slot.test.ts` and an integration test in `packages/components/src/modal/test/index.tsx` verify the attribute is correctly set and preserved during Modal open/close cycles.

## Contribution

Opened and merged by @ciampo with co-authorship from @mirka. The PR addresses issue #80304 and was backported to the Gutenberg RC and WordPress 7.1 beta after resolving a cherry-pick conflict. The fix was validated with unit tests, integration tests, and manual keyboard/screen reader testing.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
