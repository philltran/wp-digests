# #82987: Math: Source the LaTeX from the MathML annotation

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Math`
- **Merged:** [`6c9fad5`](https://github.com/WordPress/gutenberg/commit/6c9fad57d6a5190b4c17ed845a9b210170a2a7fd)
- **Discussion:** [#82987](https://github.com/WordPress/gutenberg/pull/82987) · 7 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Math block (`core/math`) now sources its `latex` attribute from the `<annotation encoding="application/x-tex">` element inside the saved `<math>` markup instead of from the block comment. This fixes a data-corruption bug where `wp_kses` (applied to users without `unfiltered_html`) mangled `&` into `&amp;` and stripped `<` characters in the comment-stored LaTeX, because the annotation content is already HTML-escaped in the markup and decoded by the parser, surviving kses intact. The block comment for `core/math` is now attribute-free (`<!-- wp:math -->`), and unparsable LaTeX is saved as an annotation-only `<semantics>` that browsers display as source text.

## Impact

- **Site owners / editors:** No action required for posts with successfully rendered math. Posts previously saved by a Contributor with corrupted `&amp;` in the block comment will be repaired on the next save (the annotation already holds the correct value). No front-end rendering change.
- **Plugin & theme developers:** The `latex` attribute is no longer present in the `core/math` block comment. Any code that extracts `latex` by parsing the comment (e.g., a custom migration or export tool) must now read it from the `<annotation encoding="application/x-tex">` text content inside the `<math>` element. The attribute is still available via the block API (`attributes.latex`), so standard `parse_blocks` / `get_block_content` workflows are unaffected.
- **Hosting & platform:** No configuration or migration step. The change is backward-compatible via a new v2 deprecation that handles blocks saved with an empty `<math>` (unparsable input from before this change).
- **Headless & REST consumers:** The `latex` attribute value is unchanged in the REST response; only its serialization source moved from the comment to the annotation. No schema change.

## Technical details

**`block.json`** — the `latex` attribute definition changes from a comment-sourced string to a text-sourced attribute:

```json
// before
"latex": { "type": "string", "role": "content" }

// after
"latex": { "type": "string", "source": "text", "selector": "math annotation[encoding=\"application/x-tex\"]", "role": "content" }
```

**`save.jsx`** — when `mathML` is empty (unparsable input or converter not yet loaded), the save output is now an annotation-only `<semantics>` instead of an empty `<math>`:

```html
<!-- before (no mathML) -->
<math display="block"></math>

<!-- after (no mathML) -->
<math display="block"><semantics><annotation encoding="application/x-tex">\frac{</annotation></semantics></math>
```

Browsers render an annotation as the sole child of `<semantics>` as its text content, so the source is visible on the front end.

**`edit.jsx`** — three behavioral changes:
- Replaced the `initialLatex` ref with a `useEvent`-wrapped `renderLatest` callback so the converter re-renders the current `latex` value when it loads asynchronously (handles the race where a user types before the converter resolves).
- The editor preview for an unrendered formula now shows the same annotation-only `<math>` markup as the front end, instead of a zero-width space.
- When the converter is not yet loaded and the user types, `setAttributes` now clears `mathML` alongside setting `latex`, preventing a stale render from being read back as the source.

**`deprecated.jsx`** — a new `v2` deprecation is added and checked before the existing `v1`. It uses the old comment-sourced `latex` attribute (extracted into a shared `legacyAttributes` object) and handles blocks whose `<math>` is empty (the pre-change representation of unparsable input). On save, `v2` re-serializes using the current annotation-based format. Export is now `[v2, v1]`.

**Integration fixtures** — the canonical `core__math` fixture and its serialized output drop the `latex` attribute from the block comment. Two new fixture sets are added: `core__math__deprecated-v2` (empty `<math>` with comment-sourced `latex`) and `core__math__kses-encoded` (a matrix whose comment contains `&amp;` but whose annotation holds the correct `&`).

## Contribution

Opened by @ellatrix as an alternative to #77789 (which proposed client-side decoding of the comment). The PR was authored with Claude Code after a design discussion with @ellatrix and co-authored with @dsas. In review, @ellatrix explicitly declined to add a server-side repair for blocks already saved with double-encoded `&amp;` in the annotation, noting the author can correct and re-save; a one-time mass correction was floated but not implemented. Merged as `6c9fad5`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
