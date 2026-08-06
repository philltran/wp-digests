# Design System Theming in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Andrew Duthie
- **Published:** 2026-07-31
- **Tags:** `General`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/07/31/design-system-theming-in-wordpress-7-1/](https://make.wordpress.org/core/2026/07/31/design-system-theming-in-wordpress-7-1/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 introduces foundational design system theming for the admin interface, exposing semantic design tokens as CSS custom properties and a `ThemeProvider` React component. This allows plugins and internal components to dynamically inherit color schemes, corner radius, and cursor styles without hardcoding values. The change standardizes admin UI styling and enables consistent, themeable interfaces across the dashboard and Site Editor.

## Impact

- **Plugin & theme developers:** Can enqueue the `wp-theme` stylesheet and script handles to access design tokens and the `ThemeProvider` component. No immediate migration is required, but hardcoding admin UI colors or border radii in new components should be replaced with token references.
- **Site owners & admins:** Will see consistent, themeable admin styling that respects the new design system, with no configuration needed.
- **Headless & REST consumers:** Unaffected; this change is strictly scoped to the admin and Site Editor UI.
- **Breaking changes:** None. Existing admin styles remain functional, though developers are encouraged to adopt tokens for future compatibility.

## Technical details

The change registers two new default script/stylesheet handles: `wp-theme` (CSS) and `wp-theme` (JS). The stylesheet exposes a comprehensive set of CSS custom properties prefixed with `--wpds-` (e.g., `--wpds-color-background-surface-neutral-strong`, `--wpds-border-radius-lg`) that serve as semantic design tokens. The JS handle exports the `@wordpress/theme` package, which provides the `ThemeProvider` React component. `ThemeProvider` accepts five props: `color.primary`, `color.background`, `cursor.control`, `cornerRadius` (values: `none`, `subtle`, `moderate`, `pronounced`), and `isRoot`. When rendered, it generates a harmonious color ramp from the seed colors and applies the specified token overrides to its subtree. Components in `@wordpress/ui` already consume these tokens, and the Site Editor leverages them for dark/light mode support. Developers can wrap custom admin UI sections in `<ThemeProvider>` to override defaults, or reference tokens directly in CSS.

## Contribution

The feature was developed as part of a broader admin redesign initiative, with the implementation standardizing on a CSS custom property token system paired with a React context provider. Earlier iterations explored inline style overrides, but the team settled on a centralized token architecture to ensure consistency across the dashboard and Site Editor. The dev note was reviewed by @wildworks and @tyxla, and the final merge establishes the foundational layer with plans to expand token coverage to additional admin components in subsequent releases.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
