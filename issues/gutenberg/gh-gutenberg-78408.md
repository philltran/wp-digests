# #78408: Add `Quick draft` widget

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Experimental`, `[Feature] Dashboard`
- **Merged:** [`7ffa85d`](https://github.com/WordPress/gutenberg/commit/7ffa85d1d2a187f9235ed536524b3cba3b77baac)
- **Discussion:** [#78408](https://github.com/WordPress/gutenberg/pull/78408) · 13 comments · 0 reactions

## Summary

A new `gutenberg/quick-post` dashboard widget lands under the `[Type] Experimental` flag, turning the WordPress dashboard into a lightweight post-creation entry point. The widget presents a title-and-content authoring form with inline validation, saves the result as a `draft` post via the REST API, and transitions through four distinct states: loading, an existing-draft nudge (when the current user already created a draft today), the form itself, and a post-saved confirmation. Content entered in the plain textarea is serialized into one or more `core/paragraph` blocks before saving, so drafts open in the block editor without falling back to a Classic block. This widget is also the reference implementation for `@wordpress/dataviews`-backed form widgets on the new dashboard surface.

## Impact

**Plugin and theme developers**
- No breaking changes. No existing APIs are modified or deprecated.
- Developers building custom dashboard widgets can treat this widget as the canonical pattern for `<DataForm>`-based form UIs: `field.isValid` validation rules, `useFormValidity`, and `saveEntityRecord` dispatched from `coreDataStore`.
- The CSS module documents three `@wordpress/components` layout gaps that required `:has()`-scoped workarounds (BaseControl flex chain, DataForm row growth, DataForm wrapper growth) — useful to know before building adjacent widgets.

**Site owners / editorial teams**
- Only visible if running the experimental Gutenberg dashboard surface and explicitly inserting the `Quick Post` widget.

**All audiences**
- No action required; the widget is opt-in and gated behind the experimental dashboard.

## Technical details

New files live under `widgets/quick-post/`.

**Registration** — `widget.json` and `widget.ts` register the widget as `gutenberg/quick-post` in the `dashboard` category.

**State machine in `render.tsx`** — A single `useSelect` drives all four states:

```ts
// loading gate
hasFinishedResolution( 'getEntityRecords', [
  'postType', 'post', { status: 'draft', author, after: <localMidnight>, orderby: 'date', order: 'desc', per_page: 1 }
] )
// resolves to existing-draft prompt when records.length > 0, otherwise form
```

Using `hasFinishedResolution` (rather than `records === undefined`) ensures the spinner clears even when the selector resolves to an empty array or encounters an error.

**Form** — `<DataForm>` from `@wordpress/dataviews` renders the fields. Validation constraints live on `field.isValid` (`required`, `minLength`). `useFormValidity(data, fields, form)` wires per-field error rendering inline.

**Save** — On submit, dispatches:
```ts
useDispatch( coreDataStore ).saveEntityRecord(
  'postType', 'post',
  { title, content, status: 'draft' }
);
```
The returned post `id` and `title` populate the confirmation view, which links to `post.php?action=edit&post=<id>`.

**Content serialization** — Plain-text content is converted to block markup before the REST call: blank-line boundaries produce separate `core/paragraph` blocks, single newlines become `<br>`, and the text is HTML-escaped. This prevents a Classic block fallback in the editor.

**Custom field component** — `fields/quick-post-content-field/` supplies a custom `Edit` wrapping `TextareaControl`. Module CSS uses `:has()` selectors to fix three layout gaps in `@wordpress/components`: BaseControl flex chain, DataForm row growth, and DataForm wrapper growth.

**Post-state components** — `components/saved-post/` and `components/existing-draft-prompt/` both extend the `EmptyState` compound from `@wordpress/ui`. `SavedPost` overrides icon styling with the success design tokens `fg-content-success`, `stroke-surface-success`, and `bg-surface-success-weak`.

Known gaps noted as follow-ups: no error/retry surface for network or permission failures; no block-based content composition (a future `<BlockEditorProvider>` integration is noted but not implemented here).

## Contribution

Authored by @retrofox and merged at commit `7ffa85d`. Co-contributors credited in the merge message are @simison and @jameskoster. The PR carries 13 discussion comments; no alternative architectural approaches are documented in the provided discussion. It advances the larger dashboard redesign initiative tracked in issue #77616.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
