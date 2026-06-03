# #70575: Prevent images from appearing squished when only one dimension is set

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @lschuyler
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Image`
- **Merged:** [`48411f0`](https://github.com/WordPress/gutenberg/commit/48411f05b98227ccbce3f8f08d42c8d0932d89a1)
- **Discussion:** [#70575](https://github.com/WordPress/gutenberg/pull/70575) · 6 comments · 0 reactions

## Summary

The `core/image` block now explicitly emits `height: auto` (or `width: auto`) in its inline style whenever only one dimension is set by the user. Previously, Gutenberg serialized only the dimension the editor user touched — e.g. `style="width:64px"` — leaving the other dimension unset and therefore vulnerable to override by theme CSS that fixed both dimensions, causing the image to render distorted. The fix is applied in both the editor render path and the `save()` output. A new v9 deprecation handles existing blocks saved before this change.

## Impact

**Plugin & theme developers**
- **Themes that apply a fixed `height` (or `width`) to `.wp-block-image img` via CSS** will now find that a block where the user set only the opposite dimension overrides that CSS property with `auto`. This is the intended behaviour and matches the browser's natural aspect-ratio preservation, but it is a rendering change for any theme relying on the old implicit fallback.
- The v9 deprecation silently migrates affected blocks on next save — no manual intervention required. Block validation will not throw errors on parse; the block is marked valid via the deprecation match, then re-serialized with `height: auto` added.
- No action required for sites where image block dimensions are either both set, both unset, or controlled via `aspectRatio`/`scale` — those code paths are unchanged.

**Hosting & platform teams running server-side block serialisation**
- If you cache or pre-render serialised block HTML, affected image blocks will produce different markup after a user re-saves; cache invalidation should be triggered on save as usual.

## Technical details

**`packages/block-library/src/image/save.js`**

The direct `width`/`height` property spread is replaced with an IIFE that builds the style object conditionally. Dimension keys are only added when `width !== undefined || height !== undefined`. Within that branch:
- `width: 'auto'` is emitted for an explicit `'auto'` string; a numeric value is coerced to `'${n}px'`; `undefined`/`null` emits nothing.
- `height: auto` is emitted whenever `height` is `'auto'`, `undefined`, or `null` — i.e. whenever the user has not set a numeric height. A numeric value is coerced to `'${n}px'`.

Before:
```js
style={ {
  ...borderProps.style,
  ...shadowProps.style,
  aspectRatio,
  objectFit: scale,
  objectPosition: ...,
  width,          // e.g. "200px"
  height,         // undefined — omitted entirely by React
} }
```

After (condensed):
```js
style={ (() => {
  const style = { ...borderProps.style, ...shadowProps.style, aspectRatio, objectFit: scale, objectPosition: ... };
  if ( width !== undefined || height !== undefined ) {
    if ( width === 'auto' )      style.width  = 'auto';
    else if ( width != null )   style.width  = typeof width === 'number' ? `${width}px` : width;
    if ( height === 'auto' || height == null ) style.height = 'auto'; // ← new
    else                        style.height = typeof height === 'number' ? `${height}px` : height;
  }
  return style;
})() }
```

**`packages/block-library/src/image/image.js`**

The same IIFE pattern replaces the `{ width, height }` spread in the editor-side `<img>` style, ensuring the live editor preview is consistent with saved output.

**`packages/block-library/src/image/deprecated.js`**

- **v9** is prepended to the export array (`[v9, v8, v7, …]`). Its `save()` reproduces the old behaviour (passes `width`/`height` directly without the `auto` injection), so the block validator can match pre-fix serialised HTML. No `isEligible` guard — removed during review as redundant.
- **v7 `migrate()` fix**: changed from unconditionally appending `'px'` (`\`${width}px\``) to guarding with `typeof width === 'number'`, preventing double-appending `'px'` on values already stored as strings (e.g. `"200px"`).

**Integration test fixtures** (`test/integration/fixtures/blocks/core__image__deprecated-v9-add-height-auto.*`):
- Input HTML: `style="width:200px"` (no height)
- Serialized output: `style="width:200px;height:auto"` — confirming the migration path.

## Contribution

Opened by **@lschuyler**. **@aaronrobertshaw** provided a detailed code review that led to two notable revisions: adding the `typeof width === 'number'` guard in `v7.migrate()` to match the existing `v6` pattern, and dropping the `isEligible` function from v9 as redundant. **@youknowriad** noticed the PR had stalled after approval and pinged for merge; **@ramonjd** re-tested and merged. **@aaronrobertshaw** noted he had intended to enable auto-merge but missed it.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
