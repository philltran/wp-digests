# #78792: Media Editor Modal: Reorder details fields so the editable regular layout fields appear at the top

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Enhancement`, `[Feature] Media`
- **Merged:** [`89189a2`](https://github.com/WordPress/gutenberg/commit/89189a2453f8f27987fd966a136cadaea41d4b88)
- **Discussion:** [#78792](https://github.com/WordPress/gutenberg/pull/78792) · 6 comments · 0 reactions

## Summary

The Details tab inside the Media Editor Modal (opened via the crop icon on an Image block) now surfaces editable fields — alt text, title, caption, description — at the top of the panel, with read-only metadata fields pushed to the bottom. Alongside the reorder, two cosmetic fixes land: date fields ("Date added", "Date modified") drop their previous custom truncation logic and render via the native `datetime` input type directly, with `text-wrap: pretty` added to prevent orphaned single-word lines; and the filename field no longer truncates inside this modal, reserving truncation for the DataViews list/grid/table contexts where column width actually constrains it.

## Impact

**Plugin & theme developers**
- No action required. This is a pure editor UI change with no public API surface, no new hooks, and no block.json or REST schema changes.

**Site owners / content editors**
- Editable metadata fields are now immediately visible on opening the Details tab; read-only file metadata (dimensions, file size, upload date, etc.) scrolls below them. Date values will no longer be truncated with an ellipsis and may wrap to a second line.

**Hosting & platform teams**
- No action required.

**Headless & REST consumers**
- No action required.

## Technical details

Changes land in two packages, identifiable from the build diff:

- **`@wordpress/media-utils`** — the heavier change, with both JS (−47 B minified) and CSS (+21–26 B minified) deltas. The Details panel component has its field array reordered so that writable fields (`alt`, `title`, `caption`, `description`) precede read-only ones. The date display fields previously applied custom rendering to avoid overflow/truncation; that override is removed. The native `<input type="datetime">` value is now rendered as-is, and `text-wrap: pretty` is added to the corresponding CSS rule to avoid single-word widow lines when a date wraps.

- **`@wordpress/editor`** — minor JS change (−94 B minified). The filename field definition is updated to apply `text-overflow: ellipsis` / truncation only when rendered inside a DataViews list, grid, or table layout, stripping the truncation constraint from the Details modal context so the full filename is always visible there.

No PHP, no REST schema, no `block.json`, no hooks or filters were added or removed.

## Contribution

Opened and merged by @andrewserong, resolving issue #78791 which flagged that read-only metadata fields were appearing above the more actionable editable fields. The PR framing explicitly invited reviewer judgment on whether full-length dates and filenames were acceptable or too verbose ("Is it okay or too much?"), suggesting the design was considered a trade-off rather than a settled default. No alternative approaches were recorded in the six-comment discussion thread. Merged at commit `89189a2`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
