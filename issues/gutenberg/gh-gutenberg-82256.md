# #82256: Widget Dashboard: enforce the policy on the staging layer for every instance operation

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`, `[Package] Widget Dashboard`
- **Merged:** [`e309543`](https://github.com/WordPress/gutenberg/commit/e3095438c7e05f0b1ab5ccc2693bb4534fa5bd37)
- **Discussion:** [#82256](https://github.com/WordPress/gutenberg/pull/82256) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/widget-dashboard` staging layer now enforces the resolved policy on every instance operation, not just `remove`. Each incoming layout is diffed against current staging, and any change the policy denies is re-asserted before it lands: a denied `move` holds the instance's index, a denied `resize` keeps its spans, a denied `edit` keeps its attributes, and a new instance of a rejected type is dropped. Previously a composed trigger writing directly to the engine's state could move, resize, or edit an instance the interface had locked.

## Impact

- **Widget Dashboard consumers / plugin developers using `WidgetDashboard.Policy`:** A `canPerform` policy is now authoritative at the state layer, not only in the UI. Custom triggers that write layouts through the internal context can no longer bypass `move`, `resize`, `edit`, or `insert` denials.
- **Non-UI writers:** Behavior differs from before only when a single call combines a reorder with a removal. The helper holds the membership-adjusted index rather than the previously sketched anchoring.
- **Policy authors:** Instance operations carry `widgetType` only while the type is registered in `widgetTypes`. If a plugin is gone or a render module failed to load, `widgetType` is absent, so type-keyed locks do not fire. Locks that must survive that case should be decided from the instance. A new instance of an unregistered type is not dropped, since there is no type to ask about.
- **Site owners / end users:** No action required.
- The package is experimental (`status-experimental` story tag), and there are no breaking public API changes.

## Technical details

**New helper: `src/utils/enforce-layout-policy.ts`** exports `enforceLayoutPolicy( { previous, next, canPerform, widgetTypes } )`, a pure function.

- Pass one walks the incoming layout, dropping denied insertions and re-asserting denied facets (`resize` spans, `edit` attributes) from the previous version of each instance.
- Pass two places move-denied survivors and denied removals at their previous index, discounted by allowed removals.
- When nothing is reassigned, the incoming array is returned by reference.

**`src/context/dashboard-context.tsx`:** `stageLayout` previously contained an inline remove-only backstop that re-inserted locked instances after the nearest surviving predecessor. It now delegates:

```ts
setStagingLayout( ( previous ) =>
	canPerform === ALLOW_EVERY_OPERATION
		? next
		: enforceLayoutPolicy( { previous, next, canPerform, widgetTypes } )
);
```

With no policy (`ALLOW_EVERY_OPERATION`), staging stays identical to what the trigger wrote. The local `canonicalize` function was extracted to `utils/canonicalize-layout` as `canonicalizeLayout`, and is used for `hasLayoutChanges` and the `commit` publish form.

**Tests and docs:** Adds `enforce-layout-policy.test.ts` (per-operation matrix, reference-preserving path, facet independence, unregistered-type cases) and five `policy.jsdom.test.tsx` tests through the real funnel and commit paths. Adds a Storybook fixture `Playground/Debug → StagingEnforcement` (`debug.story.tsx`) with per-card lock controls and "rogue" buttons that call `onLayoutChange` via `useDashboardInternalContext()`. README, `stories/policy.md`, and CHANGELOG document the enforcement rule and the `widgetType` absence contract; the README `remove` row no longer carries its own staging note.

## Contribution

Authored by @retrofox as part of the broader Widget Dashboard policy work (closes #82005, part of #79231). The description notes a deliberate departure from the originally sketched anchoring for `move`: that approach contradicted its own goal (the instance keeping its index) and conflicted with the grid's pin, since `arrayMoveWithPinned` holds the absolute index. The merged helper holds the membership-adjusted index instead. @chihsuan is credited in the props list; the recorded discussion is otherwise bot output (size, flaky-test and performance reports).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
