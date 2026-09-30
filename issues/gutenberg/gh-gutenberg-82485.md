# #82485: Widgets: carry declarative `attributes` through the widget pipeline

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] wp-build`, `[Feature] Dashboard`, `[Package] Widget primitives`
- **Merged:** [`a70637b`](https://github.com/WordPress/gutenberg/commit/a70637bd1b0c0642d804aaf20af30f98542ad874)
- **Discussion:** [#82485](https://github.com/WordPress/gutenberg/pull/82485) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

A dashboard widget's `attributes` schema now lives in `widget.json` and travels through the whole widget pipeline: the wp-build manifest, the PHP `WP_Widget_Type_Registry`, and the `/wp/v2/widget-modules` REST response. On the client, `useWidgetTypes` merges the record's attributes with any module-provided ones by `id`. Previously attributes existed only in `widget.ts`, so the server couldn't see them, a widget without a JS module couldn't declare any, and field labels bypassed `textdomain`. The core `activity`, `events`, and `hello-world` widgets now declare theirs in JSON.

## Impact

**Widget authors (experimental dashboard widgets)**
- Attribute field declarations (`id`, `type`, `label`, `elements`, `isValid` rules, `relevance`, etc.) can move from `widget.ts` into `widget.json`. `widget.ts` remains for values that need code: `example`, an attribute's `Edit` component, or an `isValid.custom` validator.
- Attribute `label`, `header`, `description`, `placeholder`, and option labels/descriptions are now translated server-side through the widget's `textdomain`.
- A widget with no JS metadata module can now declare attributes.
- Merge rule to be aware of: on a shared `id`, the record (JSON) wins per key, module-only entries are appended, and `isValid` merges rule by rule, so a module's `custom` validator survives.

**Hosts / REST consumers**
- `/wp/v2/widget-modules` items gain a read-only `attributes` field (array or null), available in `view`, `edit`, and `embed` contexts.

**Sanitization gotchas**
- Entries lacking a non-empty string `id`, or repeating one, are silently dropped. `isValid.custom` is stripped in JSON. Only real booleans are accepted for boolean keys.

**Everyone else**
- No action required. This is under `lib/experimental/` and is part of the in-progress dashboard/widget work, so the API is not stable.

## Technical details

**Build (`packages/wp-build/lib/build.mjs`, `widget-utils.mjs`)**: `collectWidgets` reads `attributes` from `widget.json`. `generateWidgetRegistry` writes it into `build/widgets/registry.php` via a generic JSON-to-PHP literal helper. Entries without a string `id` are dropped at build time.

**PHP registry (`lib/experimental/dashboard-widgets/`)**
- `WP_Widget_Type` gets a new `public $attributes = null` property.
- New `gutenberg_sanitize_widget_attributes()` in `widget-types.php` is the registration gate. Per entry it requires a unique non-empty string `id`, keeps string keys (`type`, `label`, `header`, `description`, `placeholder`), boolean keys (`readOnly`, `isDisabled`, `enableSorting`, `enableHiding`, `enableGlobalSearch`), `elements` as `value`/`label`/`description` triples (scalar or null values), `filterBy`, `format`, `isValid` minus `custom`, `Edit` (string or non-empty array), and `relevance` restricted to `high|medium|low`. Empty values drop the key, not the entry. Returns null if nothing survives.
- `gutenberg_translate_widget_metadata()` now includes `attributes` in the fields run through `translate_settings_using_i18n_schema()`; `widget-i18n.json` gains an `attributes` schema covering label, header, description, placeholder, and `elements[].label`/`description`.

**REST (`class-wp-rest-widget-modules-controller.php`)**: `prepare_item_for_response()` emits `attributes` when `rest_is_field_included()`. The item schema declares it as an array of objects with `type` as an open string and `relevance` as an enum.

**Client (`packages/widget-primitives`)**: new `WidgetAttributeRecord` type joins `WidgetModuleRecord`. `useWidgetTypes` resolves record attributes both for records without a module (schema supplied for the first time) and with one (merge by `id`, record wins shared keys, module-only entries appended, `isValid` merged per rule). Named field types (`"type": "location"`) are still resolved via `registerFieldType()` at that boundary.

```json
// widgets/activity/widget.json (illustrative shape)
{
  "attributes": [
    { "id": "per_page", "type": "integer", "label": "Items per page" }
  ]
}
```

The `activity` and `events` widgets drop `widget.ts` entirely; `hello-world` keeps it for `example`. Bundle size for `widget-primitives/index.min.js` grows by 107 B. Docs (architecture doc, package READMEs, field-types story) are updated.

**Noted follow-ups**: add `attributes` to `schemas/json/widget.json` once #77640 lands; drop the leftover `@wordpress/widget-primitives` dependency from the three widgets' `package.json` (needs a lockfile update); extend gettext extraction (#80564) to the attribute paths.

## Contribution

The record shows a single-author PR by @retrofox, with @chihsuan credited via co-authorship. It closes #82483 and is part of the broader widgets effort in #77629, building on the field type registry from #80148. The only discussion is the author syncing a few leftover doc references after initial submission; CodeRabbit review was skipped. No alternative designs are recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
