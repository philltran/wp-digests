# #80278: UI: Hook Popover into the wp compat overlay slot

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Package] UI`
- **Merged:** [`2c193dc`](https://github.com/WordPress/gutenberg/commit/2c193dc0e61863447685b337441753b7dd6100b3)
- **Discussion:** [#80278](https://github.com/WordPress/gutenberg/pull/80278) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/ui` `Popover` component now defaults its portal container to the WP compat overlay slot when the host environment opts in. This resolves a stacking issue where controlled popovers rendered inside `@wordpress/components` Modals or Popovers would appear beneath the host overlay. The change preserves explicit `container` prop overrides and keeps `Popover` marked as `use-with-caution`.

## Impact

- **Plugin & theme developers using mixed libraries:** If you render a `@wordpress/ui` `Popover` inside an `@wordpress/components` `Modal` or `Popover`, the popup will now reliably stack above the host overlay when the compat slot is enabled.
- **No action required** for standard `@wordpress/components`-only usage.
- **Opt-in required:** The behavior activates only when the WP compat overlay slot is enabled (typically via the `useEnableWpCompatOverlaySlot` hook or core's compat layer). Explicit `Popover.Portal` `container` props continue to take precedence.

## Technical details

- In `packages/ui/src/popover/portal.tsx`, the `PopoverPortal` component now destructures `container` from props and passes `container={ container ?? getWpCompatOverlaySlot() }` to the underlying `_Popover.Portal`.
- The fallback `getWpCompatOverlaySlot()` is imported from `packages/ui/src/utils/wp-compat-overlay-slot`.
- The compat slot is activated opt-in via the `useEnableWpCompatOverlaySlot` hook, which sets a runtime flag checked by the portal logic.
- Unit tests in `packages/ui/src/popover/test/index.test.tsx` verify three states: slot usage when opted in, dormant default when not opted in, and explicit `container` prop override.
- A Storybook fixture (`storybook/stories/playground/wp-compat-overlay-slot.story.jsx`) demonstrates the controlled `@wordpress/ui` `Popover` nested inside `@wordpress/components` overlays, including a nested `Autocomplete`.

## Contribution

Opened by @ciampo as a follow-up to #77851, this PR was merged as `2c193dc` and backported to both the Gutenberg RC and WordPress Core. The discussion was brief, focusing on test coverage for the opt-in, dormant, and explicit-container states. The author noted that keeping `Popover` unrecommended preserves the existing caveat against arbitrary mixed-overlay descendants while enabling controlled uses to participate in the compat mechanism.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
