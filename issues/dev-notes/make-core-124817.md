# Editable blocks inside the Custom HTML block

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Riad Benguella
- **Published:** 2026-07-23
- **Tags:** `General`, `7.1`, `blocks`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/07/23/editable-blocks-inside-the-custom-html-block/](https://make.wordpress.org/core/2026/07/23/editable-blocks-inside-the-custom-html-block/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 enables the `core/html` block to contain editable inner blocks alongside static HTML markup. When rendered in the editor, the static HTML remains inert while inner blocks are editable in place but locked to prevent structural changes. This change also introduces an `innerContent` field to `wp.blocks.registerBlockVariation`, allowing developers to define fixed markup shells with designated editable slots for `core/html` variations without building a custom block.

## Impact

- **Plugin & theme developers:** Can now create reusable `core/html` variations with embedded editable slots using the new `innerContent` array field. No custom block registration or build step is required.
- **Site owners & content editors:** Can mix static HTML with editable blocks in a single `core/html` block. Inner blocks are locked to their positions, preventing accidental structural edits while preserving the surrounding markup.
- **AI tool integrations:** Generated `core/html` blocks with mixed static/editable markup are now immediately safe and functional in the editor.
- **No action required** for existing sites or content; serialization round-trips remain unchanged.

## Technical details

The `core/html` block now parses and renders inner block markup interleaved with static HTML. In the editor, inner blocks are rendered as editable but locked components, preventing drag-and-drop reordering, removal, or sibling insertion. The block's serialization logic preserves the exact markup structure, ensuring backward compatibility.

Additionally, `wp.blocks.registerBlockVariation` now accepts an `innerContent` property specifically for `core/html` variations. This field takes an array of static HTML strings, where `null` values mark the insertion points for corresponding `innerBlocks` entries. Example usage:

```js
wp.blocks.registerBlockVariation( 'core/html', {
  name: 'testimonial-card',
  title: 'Testimonial Card',
  icon: 'format-quote',
  innerContent: [ '<div class="testimonial-card">', null, '</div>' ],
  innerBlocks: [
    [ 'core/paragraph', { content: 'An inspiring quote.' } ],
  ]
} );
```

The `innerContent` field is strictly scoped to `core/html` variations and is ignored for other block types.

## Contribution

The published dev note contains no discussion detail regarding the design debate or review timeline.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
