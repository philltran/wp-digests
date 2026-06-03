# #78402: Grid: don't allow resizing tile beyond min row height or column width

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @simison
- **Labels:** `[Type] Experimental`, `[Feature] Dashboard`
- **Merged:** [`03836ff`](https://github.com/WordPress/gutenberg/commit/03836ff2c872fc718b9e859984f31fde31609b7c)
- **Discussion:** [#78402](https://github.com/WordPress/gutenberg/pull/78402) · 8 comments · 1 reactions

## Summary

The Grid package now enforces size clamping on dashboard tiles, preventing manual resizing from shrinking beyond a single column width or row height. This change addresses layout stability in the Dashboard lanes editor by ensuring widgets remain within structurally viable dimensions during interaction.

## Impact

- **Dashboard & block developers:** Tile resize handlers in `packages/grid/src/dashboard-lanes/lanes-item.tsx` now clamp minimum dimensions. Custom tile implementations relying on unconstrained shrinking will see layout bounds enforced automatically.
- **Frontend/PHP consumers:** No impact; changes are isolated to the editor UI.
- **Action required:** None for end users or site owners. Developers may want to verify that downstream Cypress/UI tests account for the new size clamping boundaries.

## Technical details

- The diff modifies resize interaction logic in `packages/grid/src/dashboard-lanes/lanes-item.tsx` to calculate target dimensions against the current single column width and row height, applying a hard clamp before state updates.
- The behavior replaces unbounded shrinkage with structural bounds, ensuring tiles cannot be dragged below the baseline metric for a single grid lane.
- Visual feedback at the clamp boundary (e.g., rendering a border when resizing hits the minimum edge) was explicitly considered by reviewers but deferred to a follow-up PR due to complexity around handling mixed-axis invalidity.

## Contribution

- Opened by @simison as part of issue #77616 for Dashboard grid improvements.
- Review by @retrofox and @jameskoster focused on UI polish at clamp boundaries; the team agreed to ship clamping logic immediately and defer visual feedback (border rendering) to a subsequent iteration.
- Merge conflicts were resolved programmatically via an automated coding agent. Merged in commit `03836ff`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
