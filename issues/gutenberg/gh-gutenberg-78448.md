# #78448: UI: Update `@base-ui/react` to `1.5.0`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`e7f7f56`](https://github.com/WordPress/gutenberg/commit/e7f7f56a74f958e1131ee235e18fe86fcde2ea84)
- **Discussion:** [#78448](https://github.com/WordPress/gutenberg/pull/78448) · 7 comments · 0 reactions

## Summary

The `@wordpress/ui` package updated its `@base-ui/react` dependency from `1.4.1` to `1.5.0`, incorporating upstream bug fixes, accessibility improvements, and performance work. The only consumer-visible behavioral shift occurs in the `Tabs` component: in uncontrolled mode, `onValueChange` now fires when Base UI automatically selects a tab (e.g., initial mount without `defaultValue` or fallback to the first enabled tab). This aligns WordPress's wrapper behavior with the upstream fix for mui/base-ui#2097.

## Impact

- **Plugin & theme developers**: Minor behavioral shift in `Tabs` when used in uncontrolled mode. If you rely on `onValueChange` firing exclusively for explicit user interactions, be aware it now also triggers during automatic fallback selections.
- **Hosting & platform teams**: Standard dependency bump with no required configuration changes or server-side updates.
- **Migration note**: The upstream breaking change in OTP Field (`sanitizeValue()` → `normalizeValue()`) does not affect WordPress since the component is not wrapped. No code migration or test updates are strictly required beyond validating existing `Tabs` flows.

## Technical details

- **Dependency update**: `packages/ui/package.json` now requires `@base-ui/react@^1.5.0`, which pulls in `@base-ui/utils@0.2.9`. Lockfile refreshed accordingly.
- **`Tabs` test assertions**: `packages/ui/src/tabs/test/index.test.tsx` was extensively updated to verify the new callback timing. Uncontrolled tests now assert `mockOnValueChange` is called once (or twice, when a fallback occurs) during automatic selection. Controlled tests were converted to explicitly assert `onValueChange` does **not** fire for consumer-provided initial values.
- **CSS cleanup**: Removed a temporary workaround rule in `packages/ui/src/drawer/style.module.css` (`[data-wpds-theme-provider-id]:has(> .popup) { display: block; }`) tied to a resolved Base UI tabbability regression, along with its enclosing comment.
- No new public APIs, hooks, or REST routes were introduced; the change is primarily a dependency upgrade with synchronized test assertions.

## Contribution

Open and merged by @ciampo (co-authored with @aduth and @michaldudak). The author initially proposed deferring the merge to allow `1.5.0` to settle following previous release experiences. Maintainer @aduth reviewed upstream regression reports (`#4866`, `#4925`, `#4893`, etc.) and confirmed via component analysis that none of the reported production-only or store-split issues affect WordPress's usage of Base UI components. The PR was merged without delay after reviewers verified alignment with upstream behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
