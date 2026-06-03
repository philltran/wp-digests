# #78465: Dashboard: per-instance widget settings drawer

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Experimental`, `[Feature] Dashboard`
- **Merged:** [`3b9396a`](https://github.com/WordPress/gutenberg/commit/3b9396ad8fc5cbc352753f64af92c91d4f9693e4)
- **Discussion:** [#78465](https://github.com/WordPress/gutenberg/pull/78465) · 10 comments · 0 reactions

## Summary

Closes the loop for configuring widget instance attributes in the experimental dashboard by introducing a per-instance settings gear to each widget tile. The change adds an interactive UI control that opens a side drawer to edit declared `attributes`, allowing site managers to configure widgets without modifying code or editing block templates directly.

## Impact

- **Dashboard builders & theme/plugin authors:** Widget types declaring an `attributes` schema now have a native, declarative UI for instance configuration. Plugin or theme authors demonstrating new widget types can expect instances to be editable via gear icon without additional scaffolding.
- **No breaking changes.** The feature is gated behind the `gutenberg-dashboard-widgets` experiment and does not affect the classic widgets screen or existing REST API contracts.
- **Action required:** None for standard installations. Developers relying on the experimental dashboard should verify that custom widget types declare an `attributes` schema to expose the new drawer affordance.

## Technical details

The PR introduces a shared `WidgetToolbar` shell supporting two modes: `WidgetSettingsToolbar` (gear control) and `WidgetLayoutToolbar` (width menu + remove). The gear is rendered in the grid's `actionableArea` slot, remaining interactive even when the dashboard chrome is marked inert during layout edits.

- A single `WidgetSettings` drawer is mounted once at the dashboard root. Clicking the gear writes the target instance's UUID to the dashboard UI context; the root drawer reacts, reads the widget type's `attributes` schema, and renders a `<DataForm>`.
- Edits flow through a staging buffer: changes are live-previewed behind the drawer, **Save** explicitly commits alongside layout updates via `setAttributes`, and any other dismissal reverts to the staged state.
- Visibility is handled via CSS `opacity` rather than `display:none` to keep the gear in the tab order for keyboard/screen-reader users. It reveals on hover or `:focus-within` and offsets past the fixed admin menu when opening from the inline-start edge.
- The "Hello World" example widget is updated to include a `message` attribute, providing an end-to-end demonstration of the dataform flow.

## Contribution

The change was merged into the Gutenberg repository by @retrofox, with co-authorship from @simison and @jameskoster. During review, Copilot resolved merge conflicts against trunk, preserving both new styling imports and the PR's attribute rendering logic. The pull request closed without further discussion, with follow-up tasks noted for adding horizontal dashboard padding to support wider grid layouts.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
