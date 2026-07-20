# #80148: Widget Primitives: Add a field type registry for widget attributes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`, `[Package] Widget primitives`
- **Merged:** [`c16a152`](https://github.com/WordPress/gutenberg/commit/c16a152b4d1033acc337113ed93b8a08fe0290b0)
- **Discussion:** [#80148](https://github.com/WordPress/gutenberg/pull/80148) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Introduces a field type registry to `@wordpress/widget-primitives` that allows widget attributes to reference reusable, named field types instead of embedding component references directly in their metadata. This decouples widget schemas from UI controls, enabling declarative attribute definitions while letting the host application resolve and render the appropriate DataViews field controls.

## Impact

- **Plugin & theme developers:** Widget metadata can now use plain string `type` values instead of importing or referencing `Edit` components directly. Schemas become fully serializable and reusable across widgets.
- **Dashboard/Widget host developers:** Must register field types via `registerFieldType()` before rendering widgets if they want custom controls to resolve. Unregistered names degrade gracefully to standard DataViews behavior.
- **No action required** for existing widgets that don't use custom field types; they continue to work as before.

## Technical details

- New module: `packages/widget-primitives/src/field-types/field-types.ts`
- Public API: `registerFieldType( definition )`, `unregisterFieldType( name )`, `getFieldType( name )`, `resolveFields( fields )`
- Types: `FieldTypeDefinition`, `ResolvableField`, `RegisteredFieldTypeName`
- `resolveFields()` iterates over widget attributes, matches `type` against the registry, and merges definition defaults (`Edit`, `isValid`, `baseType`, etc.) into the field object. The resolved `type` becomes the `baseType` from the definition. Field-level props override definition defaults.
- Names follow a lowercase alphanumeric pattern with optional namespace (`acme/rating`). First registration wins; invalid or duplicate names are ignored. Unregistered names pass through unchanged.
- Resolution is applied internally via the `useWidgetTypes` hook.
- Before/after pattern:
  ```ts
  // Before: schema binds to code
  { id: 'location', Edit: LocationControl }
  
  // After: schema is declarative, resolved by host
  { id: 'location', type: 'location' }
  ```
- The dashboard route registers the `location` field type, and the events widget is migrated to consume it by name.

## Contribution

Opened and merged by @retrofox with co-authorship from @chihsuan. The PR introduces the registry, adds unit tests, updates the changelog and README, and includes a Storybook story demonstrating the pattern. Review focused on documentation alignment, with follow-up work planned to move attribute manifests to `widget.json` and expose the registry as a shared script module for third-party extensibility.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
