# #82527: DataForm: Add a `showPlaceholderIfEmpty` option to the panel layout

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`473906e`](https://github.com/WordPress/gutenberg/commit/473906ec3a066f8944bb036fdbdebbe8f55cf8c6)
- **Discussion:** [#82527](https://github.com/WordPress/gutenberg/pull/82527) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `panel` layout in `@wordpress/dataviews` DataForm gains a `showPlaceholderIfEmpty` boolean option. When `true`, the panel summary renders the field's `placeholder` in place of the field's `render` output if the value is `undefined`, `null`, or an empty string. It defaults to `false`, so existing forms are unchanged. The option is also added to the view-config REST schema for the panel form layout.

## Impact

**Plugin & theme developers using DataForm**
- Opt-in only; no action required and no breaking changes. Existing panel forms render exactly as before.
- To get an "Add a title"-style empty state in a panel row, set `showPlaceholderIfEmpty: true` on the panel layout and give the field a `placeholder`.
- If a field has no `placeholder`, the summary falls back to the field's `render` output even with the option on.

**Headless & REST consumers**
- The view config controller schema (7.2) now accepts `showPlaceholderIfEmpty` (boolean) on panel form layouts.

**Site owners / hosting**
- No action required.

## Technical details

**Types and normalization**
- `PanelLayout` (`packages/dataviews/src/types/dataform.ts`) gets `showPlaceholderIfEmpty?: boolean`; `NormalizedPanelLayout` gets a required `showPlaceholderIfEmpty: boolean`.
- `normalizeLayout` in `dataform-layouts/normalize-form.ts` sets `showPlaceholderIfEmpty: layout?.showPlaceholderIfEmpty ?? false`.

**Rendering**
- `panel/summary-button.tsx` introduces an internal `SummaryValue` component, used for both the multi-summary and single-summary branches in place of `<summaryField.render />`.
- `SummaryValue` renders `<span className="dataforms-layouts-panel__field-placeholder">{ field.placeholder }</span>` when `showPlaceholderIfEmpty` is on, `field.placeholder` is truthy, and `field.getValue( { item } )` is one of `undefined`, `null`, `''`. Otherwise it renders `field.render`.
- Per the code comment, emptiness mirrors the `required` validator; fields whose empty value differs (e.g. id `0`, objects) are expected to normalize that in `getValue`.
- The edit trigger (pencil/dropdown) is unaffected.

**PHP / REST**
- `Gutenberg_REST_View_Config_Controller_7_2` overrides `get_form_layout_schema()`, walks `oneOf`, and adds `showPlaceholderIfEmpty => array( 'type' => 'boolean' )` to the layout whose `type.enum` is `array( 'panel' )`.
- A `backport-changelog/7.2/13438.md` entry links the corresponding wordpress-develop PR #13438.

**Other**
- README documents the option; CHANGELOG entry added under Enhancements.
- Storybook `DataForm > Layout Panel` gets a `showPlaceholderIfEmpty` control, and the sample `title` field gets `placeholder: 'Add a title'`.
- Four jsdom tests cover opt-in with an empty value, a non-empty value, fallback to `render` without a placeholder, and the default-off behavior. `normalize-form` tests are updated for the new default.

```ts
form: {
  layout: { type: 'panel', showPlaceholderIfEmpty: true },
  fields: [ 'title' ],
}
// field: { id: 'title', type: 'text', placeholder: 'Add a title' }
```

## Contribution

This was a follow-up to a review comment on PR #82423. The record shows @ntsekouras as author, with @oandregal credited by props-bot. The description notes it was generated with an AI tool and manually reviewed. Automated review was skipped and the thread carries no design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
