# #78704: Media Editor Modal: Update the rotation ruler to use a vertical line marker

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Enhancement`, `[Feature] Media`
- **Merged:** [`e87b304`](https://github.com/WordPress/gutenberg/commit/e87b3048d73d59e88bbdcdb01ce0db8480a19f72)
- **Discussion:** [#78704](https://github.com/WordPress/gutenberg/pull/78704) · 3 comments · 0 reactions

## Summary

The rotation ruler in the experimental Media Editor Modal has been updated to replace the triangle position marker with a vertical line, matching design feedback from the parent tracking issue. Alongside the marker shape change, the active-degree readout has been refactored so the numeric digits are always centered directly beneath the marker regardless of whether a negative-sign prefix is present.

## Impact

- **Site owners / content editors:** Visual improvement to the fine-grained rotation control inside the Media Editor Modal crop interface; behavior is unchanged.
- **Plugin & theme developers:** No API, hook, filter, or `block.json` changes. No action required.
- **All audiences:** This change is gated behind the **Media Editor Modal** Gutenberg experiment and has no effect on sites that have not opted in.

## Technical details

All changes are in `packages/media-editor/src/components/rotation-ruler/index.tsx`.

The marker element was previously rendered as a triangle (likely a CSS `border`-trick or SVG polygon). It is now a vertical line element styled via the editor stylesheet (`build/styles/editor/style.css` / `style-rtl.css`), adding ~31–45 B to those bundles.

The active-degrees label rendering was restructured to split the string into three discrete pieces:
- **sign** — the `-` character when the value is negative, absolutely positioned to the left of the digit string
- **digits** — the numeric characters, used as the centering anchor
- **degrees symbol** — `°`, absolutely positioned to the right of the digit string

This ensures the digit block stays centered under the vertical marker line in all cases, which previously shifted when the minus sign was present. The overall `index.min.js` bundle for the editor grew by ~61 B.

## Contribution

Opened and merged by **@andrewserong** as part of the ongoing Media Editor Modal redesign tracked in #73771. The direction came from design feedback posted in that issue. **@ramonjd** is listed as a co-author. Discussion on the PR itself was minimal (3 comments, bot-generated); the design rationale was established upstream in the tracking issue.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
