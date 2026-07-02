# #79638: Expose widget category through the build pipeline and REST API

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] wp-build`, `[Package] Widget primitives`
- **Merged:** [`67550ae`](https://github.com/WordPress/gutenberg/commit/67550ae477c5dcfb4cf12483d1f58823d205e041)
- **Discussion:** [#79638](https://github.com/WordPress/gutenberg/pull/79638) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The experimental widget type contract now propagates a `category` field end-to-end. Previously declared only in `widget.json`, the field is now carried through the PHP registry, exposed via the `/wp/v2/widget-modules` REST endpoint, and surfaced in the JavaScript primitives hook. This enables hosts and UI consumers to group or filter widget types by category without importing individual metadata modules.

## Impact

- **Plugin & theme developers**: No breaking changes or immediate migration required. Existing `widget.json` files will continue to work; adding a `category` string opt-in enables grouping in client-side pickers.
- **Dashboard hosts & platform teams**: The `/wp/v2/widget-modules` REST schema now includes a read-only `category` field (`string|null`). Hosts querying this endpoint can filter or organize widgets server-side or client-side without loading the full metadata module surface.
- **Backward compatibility**: Fully additive. Registry arrays, `WP_Widget_Type` properties, and REST responses remain structurally compatible with prior releases.

## Technical details

- `packages/wp-build/lib/build.mjs`: `collectWidgets()` now reads `metadata.category` from each `widget.json` and injects it into the generated registry array via `generateWidgetRegistry()`.
- `lib/experimental/dashboard-widgets/widget-types.php`: `gutenberg_register_widget_types()` maps `$widget['category'] ?? null` into the new property during registration.
- `lib/experimental/dashboard-widgets/class-wp-widget-type.php`: Adds a public `$category = null;` property to the `WP_Widget_Type` class.
- `lib/experimental/dashboard-widgets/class-wp-rest-widget-modules-controller.php`: Extends `get_item_schema()` with a `category` field definition (`type: ['string', 'null']`, `readonly: true`) and conditionally includes it in `prepare_item_for_response()` using `rest_is_field_included('category', $fields)`.
- `packages/widget-primitives/src/hooks/use-widget-types.ts`: Merges `record.category` into the resulting `WidgetType` object, overriding the metadata module's value if present (mirroring existing `presentation` overlay logic).
- `packages/widget-primitives/src/types.ts`: Adds `category?: WidgetTypeMetadata['category'] | null;` to the `WidgetModuleRecord` interface.

## Contribution

Open by @retrofox as part of the broader widget schema roadmap (#77629). Co-authored and merged by @simison after internal review. The implementation deliberately mirrors the existing `presentation` propagation pattern to maintain consistency across the build pipeline, REST API, and JS primitives without introducing new hooks or altering registration behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
