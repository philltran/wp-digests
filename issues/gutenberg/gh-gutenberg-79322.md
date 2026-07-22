# #79322: Widget Primitives: decouple discovery from a hardcoded endpoint

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] Widget primitives`
- **Merged:** [`fb05ad0`](https://github.com/WordPress/gutenberg/commit/fb05ad08bb753d823df192aefd5ecb836b834903)
- **Discussion:** [#79322](https://github.com/WordPress/gutenberg/pull/79322) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/widget-primitives` package decouples widget type discovery from the WordPress data layer by making `useWidgetTypes()` accept host-supplied records instead of reading from a hardcoded `widgetModule` core-data entity. This change removes the package's dependency on `@wordpress/core-data` and `@wordpress/data`, allowing the hook to operate in any React application. The shift aligns discovery with the existing `resolveWidgetModule` pattern, ensuring the package remains truly host-agnostic.

## Impact

- **Host implementers & plugin/theme developers:** Breaking change to the `useWidgetTypes()` hook signature. It now requires a `records` argument (`WidgetModuleRecord[] | null | undefined`) instead of fetching data internally. Hosts must fetch widget-module records via their preferred data source and pass them to the hook.
- **Widget authors:** No direct impact; contract types (`WidgetType`, `WidgetRenderProps`, etc.) remain unchanged.
- **Hosting & platform teams:** The package no longer registers the `widgetModule` core-data entity on import. Any host relying on this side-effect must explicitly register the entity and supply records.
- **Migration:** Update calls to `useWidgetTypes()` to pass the fetched records array. Remove any reliance on the automatic `widgetModule` entity registration.

## Technical details

The diff modifies `packages/widget-primitives/src/hooks/use-widget-types.ts` to remove `dispatch`, `useSelect`, and `coreStore` imports. The hook signature changes from `useWidgetTypes(): UseWidgetTypesResult` to `useWidgetTypes(records: WidgetModuleRecord[] | null | undefined): UseWidgetTypesResult`. The internal `WidgetModuleRecord` interface is promoted to a public export via `packages/widget-primitives/src/index.ts`. The package’s `package.json` drops `@wordpress/core-data`, `@wordpress/data`, and `@wordpress/i18n` from dependencies. Hosts like the dashboard engine (`routes/dashboard/route.ts`) now handle entity registration in their `beforeLoad` phase and pass records to the hook.

Before/after usage pattern:
```diff
-const [widgetTypes, isResolving] = useWidgetTypes();
+const records = useSelect((select) => select(coreStore).getEntityRecords('root', 'widgetModule'));
+const [widgetTypes, isResolving] = useWidgetTypes(records);
```

## Contribution

Opened by @retrofox to address host-coupling concerns raised in #79309. During review, @louwie17 questioned whether the primitives should couple with `core-data` to fetch widget types, prompting @retrofox to refactor the implementation so the host owns data retrieval entirely. The approach was refined to accept `null` or `undefined` during loading states, and the PR was merged with co-authorship from @louwie17 and Copilot. The change remains scoped to the experimental dashboard widgets pipeline.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
