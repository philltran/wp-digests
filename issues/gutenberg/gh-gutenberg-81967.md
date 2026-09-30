# #81967: Widget Dashboard: add a Policy provider to govern what users may do

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`, `[Package] Grid`, `[Package] Widget Dashboard`
- **Merged:** [`07a4a06`](https://github.com/WordPress/gutenberg/commit/07a4a0632117f23ff1bc3a328cccad899f8c08e2)
- **Discussion:** [#81967](https://github.com/WordPress/gutenberg/pull/81967) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds `WidgetDashboard.Policy`, a provider that governs every dashboard mounted below it through a single callback, `canPerform( request )`. The request names one of six operations: `customize` and `insert` at the dashboard level, and `remove`, `move`, `resize`, and `edit` per placed widget instance. Hosts can now map their own user capabilities to dashboard permissions in one place, instead of relying on `editMode` (all-or-nothing) or filtering `widgetTypes` (which also stops placed widgets from rendering). `@wordpress/grid` gains per-item `draggable` and `resizable` layout flags to support the `move` and `resize` operations.

## Impact

**Plugin/theme developers and hosts embedding `@wordpress/widget-dashboard`**
- New opt-in API: wrap `<WidgetDashboard>` in `<WidgetDashboard.Policy canPerform={...}>`. Without a policy every operation is allowed, so existing integrations behave as before.
- Nested policies only narrow permissions; they cannot re-grant what an outer policy denied.
- Return `true` for operations you do not govern. The operation vocabulary may grow, so a `default: return true` branch is the forward-compatible pattern.
- The policy governs the **interface only**, not what the server accepts. Persistence-side authorization (e.g. a `403` after `onLayoutChange`) remains the consumer's job.

**`@wordpress/grid` consumers**
- Layout items in `DashboardGrid` and `DashboardLanes` accept `draggable` and `resizable` booleans (default `true`). A `draggable: false` item is pinned and holds its index while the others reorder.

**Site owners / end users**
- No visible change. The WordPress dashboard route does not mount a policy yet, so the built dashboard behaves as before. Mounting one (layout open, `edit` mapped to the `edit_dashboard` capability) is a listed follow-up.

**Known gap**
- Only `remove` has a staging-layer backstop. Re-asserting `move`, `resize`, and `edit` denials in the staging funnel is deferred to a follow-up.

## Technical details

**Contract.** The request is a discriminated union on `operation`:

```tsx
<WidgetDashboard.Policy
	canPerform={ ( request ) => {
		switch ( request.operation ) {
			case 'customize': return canEditLayout;
			case 'insert':    return request.widgetType.category === activeSection;
			case 'remove':    return ! request.widget.attributes?.pinned;
			default:          return true;
		}
	} }
>
	<WidgetDashboard { ...props } />
</WidgetDashboard.Policy>
```

`customize` carries no subject; `insert` carries `widgetType`; `remove | move | resize | edit` carry `widget` and `widgetType`.

**Resolution.** The dashboard provider reads the policy and exposes a resolved `canPerform` in its internal context. Every surface queries that single point, so future sources (the instance `lock` datum in #81900, an extension filter `widgetDashboard.canPerformWidgetOperation`) can compose there. `Policy` must wrap `<WidgetDashboard>` because the engine mounts the inserter outside `children`.

**Enforcement (per the PR description; the widget-dashboard source diff was truncated in the provided input):**
- `insert` filters the picker; the Add widget button and command appear only while some type is insertable.
- `customize` gates the Customize button, the related commands, and the empty-layout auto-entry. Done and Cancel stay available while customizing.
- `remove` gates the Remove control, and the staging layer re-asserts a locked instance dropped by any trigger.
- `resize` gates the width menu and resize handle; `move` gates the drag gesture, both via the new grid flags.
- `edit` gates inline fields, the settings trigger and surface, and the widget's `setAttributes`, so a denied widget renders read-only through its existing contract.

**`@wordpress/grid` (from the diff):**
- `DashboardGridLayoutItem` and `DashboardLanesLayoutItem` gain `draggable?: boolean` and `resizable?: boolean`.
- `GridItem` and `LanesItem` compute `dragDisabled = disabled || ! draggable` (passed to `useSortable({ disabled })` and the cursor) and `resizeDisabled = disabled || ! resizable` (controls rendering of `ResizeHandle`).
- New `src/shared/array-move-with-pinned.ts` exports `arrayMoveWithPinned( items, from, to, isPinned )`. It runs `arrayMove`, then refills non-pinned slots in order so pinned items keep their index; moving a pinned item is a no-op. `DashboardGrid` and `DashboardLanes` use it in place of `arrayMove` and now return early when the resulting order is unchanged.
- Tests added for both components' item interactions and for `arrayMoveWithPinned`.

**Other:** `@wordpress/admin-ui` is added as a dev dependency of the widget-dashboard package (`package-lock.json`) for the Storybook Policy story. Docs are added: a README *Governance* section, a Storybook *Policy* page, and a line in `docs/explanations/architecture/dashboard-widgets.md`. The grid CHANGELOG records the new flags.

## Contribution

Opened by @retrofox as a follow-up to review on #81893, closing #81892, delivering the host seam of #81900, and part of #79231. A reviewer asked whether the policy enforces every layout mutation or mainly controls the UI. @retrofox answered that it governs the interface only, since the engine is agnostic of data and persistence, but agreed the engine should be consistent with its own policy, because only `remove` had a staging-layer backstop. Extending that to `move`, `resize`, and `edit` was split into a follow-up (#82005) to keep this PR's size. Four of five review threads landed as commits before merge; @chihsuan is credited in the props list.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
