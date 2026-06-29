# #79554: Tab List: Fix render inline formatting on frontend

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @prasadkarmalkar
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Tabs`
- **Merged:** [`85ec664`](https://github.com/WordPress/gutenberg/commit/85ec66417963e0cd764d3641daf58ab1d4a1f1b1)
- **Discussion:** [#79554](https://github.com/WordPress/gutenberg/pull/79554) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Resolves a frontend rendering bug in the core Tab List block where inline formatting (bold, italic, line breaks) applied to tab labels was outputting raw HTML tags as visible text. The update modifies the block's metadata schema and save routine to properly decode and render rich-text markup on the live site.

## Impact

- **Core/Block Developers:** The `label` attribute for tabs now expects and handles rich-text input, aligning with how formatted content is stored and rendered.
- **Site Owners & Editors:** Applying bold or italic styles to tab labels in the block editor will no longer result in escaped HTML tags appearing on the front end.
- **Migration/Configuration:** No immediate action required for existing installations. Custom blocks or themes duplicating this `block.json` structure may need to update their attribute definitions to adopt the `rich-text` source/type if they maintain similar label attributes.

## Technical details

The fix modifies two files in `packages/block-library/src/tab-list/`:

1. **`block.json`**: Updates the `query` configuration for the `label` attribute.
```json
"label": {
    "type": "rich-text",
    "source": "rich-text",
    "role": "content"
}
```
This replaces the previous `"type": "string", "source": "html"` configuration, explicitly telling the block engine to parse and save formatted markup rather than treating it as a raw HTML string.

2. **`save.js`**: Replaces direct JSX interpolation with the `RichText.Content` component to render the label.
```jsx
<RichText.Content
    tagName="button"
    value={ tab.label }
    className={ buttonClassName || undefined }
    style={ buttonStyle }
    type="button"
    role="tab"
/>
```
The previous implementation rendered `{ tab.label }` directly inside a standard `<button>` tag. Switching to `RichText.Content` ensures the rich-text markup is correctly processed during save.

## Contribution

Opened by @prasadkarmalkar as part of issue #79536 to address inline formatting rendering in tabs. Initial iterations explored direct saving fixes, but code review by @Mamaduka identified that treating the `label` attribute as RichText via a schema update was the cleaner, more robust approach. The change was refined through review and merged with co-authors Mamaduka and t-hamano.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
