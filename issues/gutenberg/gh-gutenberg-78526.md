# #78526: IconButton: Fix `focusableWhenDisabled` default

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Bug`, `[Package] UI`
- **Merged:** [`851494e`](https://github.com/WordPress/gutenberg/commit/851494ef568ceb26eaa80e626bbe377480d07aa4)
- **Discussion:** [#78526](https://github.com/WordPress/gutenberg/pull/78526) · 4 comments · 0 reactions

## Summary

Fixes a behavioral mismatch in `packages/ui/src/icon-button/icon-button.tsx` where `IconButton` failed to default `focusableWhenDisabled` to `true`. This omission caused disabled icon buttons to incorrectly suppress their tooltip triggers, preventing tooltips from rendering even when the underlying button remained focusable. The fix explicitly defaults the prop to `true`, aligning behavior with `Button` and restoring expected tooltip visibility in disabled states.

## Impact

- **UI/Package developers**: No breaking changes. `IconButton` now defaults `focusableWhenDisabled` to `true`, matching `Button`. Existing usages without an explicit prop will see tooltips render on disabled buttons automatically.
- **Block/Editor developers**: Implicitly benefits editor UI components relying on disabled icon buttons for context or help tooltips. No migration, configuration, or code changes required.

## Technical details

The change updates the component definition in `packages/ui/src/icon-button/icon-button.tsx` to explicitly default the `focusableWhenDisabled` prop to `true`. Previously, omitting the prop yielded `undefined`, which broke the tooltip trigger logic: `disabled={ disabled && ! focusableWhenDisabled }` evaluated to `true`, hiding the trigger. By defaulting to `true`, the expression evaluates to `false`, allowing `Tooltip.Trigger` to render and become focusable as documented. The component's internal test suite was also updated to explicitly pass `focusableWhenDisabled={ false }` for its "truly disabled" assertion, isolating the edge case.

## Contribution

Opened by @mirka and merged following review from @aduth. Discussion focused on whether both `disabled` and `focusableWhenDisabled` should be destructured at all, but the team agreed destructuring is required to pass the prop downstream to `Tooltip.Trigger`. The fix was accepted as a straightforward default-value update with no architectural changes or rejected alternatives discussed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
