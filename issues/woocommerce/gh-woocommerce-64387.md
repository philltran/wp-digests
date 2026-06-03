# #64387: Add modern settings SDK opt-in path

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @dmallory42
- **Labels:** `Documentation`, `plugin: woocommerce`, `focus: monorepo infrastructure`, `package: dependency-extraction-webpack-plugin`
- **Merged:** [`f6617bc`](https://github.com/woocommerce/woocommerce/commit/f6617bc0c967a4f14a9e18b5dbee20f61a9d4c93)
- **Discussion:** [#64387](https://github.com/woocommerce/woocommerce/pull/64387) · 7 comments · 0 reactions

## Summary

WooCommerce introduces an opt-in React-based settings SDK that enables extensions to render a modern, canonical UI while keeping `WC_Settings_Page` as the PHP migration boundary. This architectural shift separates client-side rendering from server-side persistence and asset management, providing a structured upgrade path for extensions without requiring full legacy flow rewrites.

## Impact

- **Extension developers**: Can opt into the modern SDK via an adapter pattern; existing settings pages remain unaffected until explicitly migrated.
- **Site owners / administrators**: No action required. The update is gated behind a feature flag, preserving current behavior by default.
- **Platform & hosting teams**: No direct impact unless custom admin overrides target legacy DOM structures or jQuery-dependent scripts. Migrated pages rely on hidden input serialization for form submissions, requiring any custom validation middleware to account for this mechanism.

## Technical details

- Introduces the `Automattic\WooCommerce\Admin\Settings` namespace containing a legacy settings page adapter, schema conversion utilities, scoped JS component registration, and native SDK field rendering.
- Gated by the `modern-settings` feature flag. When enabled, the reference implementation (`WC_Settings_Products`) routes through the SDK, applying a two-column section layout with bordered content panels.
- The SDK owns structure and scoped styling; extensions only supply custom React components for non-standard fields. Standard inputs map to `@wordpress/components` controls (`BaseControl`, `TextControl`, `TextareaControl`, `SelectControl`, `CheckboxControl`, `Button`).
- Client-side state is managed by the SDK, while form submissions persist data via hidden input serialization to maintain compatibility with legacy PHP handling and existing routing.
- Asset loading order enforces `wc-modern-settings-sdk` to load prior to the settings embed script when opted in.

## Contribution

PR #64387 was merged by @dmallory42 (commit `f6617bc`). The merge process passed automated pre-merge checks and multi-environment linting. Discussion was minimal, focusing on architectural scoping rather than implementation debates; the PR description defines the boundary between SDK-owned structure/styling and extension-provided custom components without introducing alternative approaches.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
