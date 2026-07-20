# #80007: Document widget relevance, help

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] Widget primitives`
- **Merged:** [`6c23cf4`](https://github.com/WordPress/gutenberg/commit/6c23cf4d7efd340e206de9cd2dd7c68a3e826ec4)
- **Discussion:** [#80007](https://github.com/WordPress/gutenberg/pull/80007) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request audits and updates the widget system documentation to standardize host-agnostic terminology and formally document two metadata fields: the `relevance` hint on widget attributes and the `help` note in `widget.json`. It adds a Storybook story demonstrating the prominent vs. settings surface split and updates inline comments to reflect the new vocabulary. This clarifies how hosts should render widget controls and metadata without tying the contract to a specific UI implementation.

## Impact

- **Widget authors**: Can now explicitly declare `relevance: 'high'` on `WidgetAttributeField` to signal which attributes should appear inline, and add a `help` object (`content` + optional `links`) to `widget.json` for contextual tooltips.
- **Host/dashboard implementers**: Should update UI rendering logic to respect the `relevance` hint (prominent surface for `'high'`, settings surface for others) and surface `help` content in infotips. No breaking changes; existing widgets continue to work.
- **REST consumers**: The `/wp/v2/widget-modules` endpoint now includes `help` in its response payload alongside `title`, `description`, and `keywords`.
- **No action required** for existing codebases; this is a documentation and terminology update with zero functional behavior changes.

## Technical details

- Updates the `WidgetAttributeField` type to include an optional `relevance` hint (`'high' | 'low'`).
- Documents `widget.json` supporting a `help` object with `content` and optional `links`, localized via `textdomain`.
- The `WP_REST_Widget_Modules_Controller` now returns `help` in its `/wp/v2/widget-modules` records.
- `useWidgetTypes( records )` merges `help` from `widget.json` alongside other translatable strings.
- Terminology in `packages/widget-dashboard/src/components/widget-attribute-controls/widget-attribute-controls.tsx`, `widget-settings-trigger.tsx`, `widget-settings.tsx`, and `widgets.tsx` is updated to use "prominent surface" and "settings surface" instead of "inline" and "drawer".
- Adds a `WithRelevance` Storybook story in `packages/widget-primitives/src/components/widget-render/stories/index.story.tsx` and an `attribute-relevance.svg` diagram to the anatomy docs.
- The diff contains only documentation updates, type annotations, inline comments, and Storybook examples; no runtime logic was modified.

## Contribution

Opened and merged by @retrofox with co-authorship from @chihsuan. The PR focused on aligning the widget-primitives and widget-dashboard documentation with a host-agnostic contract, adding visual aids (Storybook story, SVG diagram), and standardizing UI terminology across the codebase. Review was minimal, with the diff reflecting a straightforward documentation and comment cleanup.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
