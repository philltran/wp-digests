# What’s new in Gutenberg 23.4? (June 17, 2026)

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** ramonopoly
- **Published:** 2026-06-17
- **Tags:** `Feature Plugins`, `plugins`, `block-editor`, `core-editor`, `gutenberg`, `gutenberg-new`
- **Link:** [https://make.wordpress.org/core/2026/06/17/whats-new-in-gutenberg-23-3-03-jun-2/](https://make.wordpress.org/core/2026/06/17/whats-new-in-gutenberg-23-3-03-jun-2/)
- **Usefulness:** 4/5

## Summary

Gutenberg 23.4 introduces an experimental flag to validate plugin and editor integrations against the React 19 runtime, enables block transforms to target specific block variations (e.g., converting Columns or Gallery to a Grid layout), and synchronizes the Site Editor chrome with the active admin color scheme. The release also adds UltraHDR JPEG detection, client-side media upload progress indicators, and exposes a filterable API for DataViews entity view configuration.

## Impact

- **Plugin & theme developers**: Should test React-based admin or editor integrations against the new React 19 experimental flag to catch runtime incompatibilities. Console warnings and errors are the primary indicators of breaking changes.
- **Block editors**: Transforms can now target a specific variation of another block, simplifying layout conversions without losing inner content. The Site Editor sidebar and page shell will respect the user's admin color scheme instead of a fixed dark background.
- **Media & content workflows**: UltraHDR JPEG detection preserves originals and applies ISO 21496-1 gain maps during resizing. Client-side uploads now feature a progress snackbar and automatic offline queue pause/resume.
- *No action required* for standard site updates, though verifying React 19 readiness is recommended for heavy editor extensions.

## Technical details

- **React 19 Experimental Flag**: Activated via the `/wp-admin/admin.php?page=experiments-wp-admin` experiments page. Registers `react`, `react-dom`, and `react-jsx-runtime` v19 scripts. A compatibility layer resolves runtime differences between v18 and v19. Developers should audit build pipelines to ensure `react/jsx-runtime` links to the externalized WordPress script rather than bundling a local copy.
- **Block Transform Targeting**: The internal transform logic now accepts a target variation parameter. This enables transformations like `core/columns` or `core/gallery` → `core/grid` (standard variation), preserving nested blocks while swapping the wrapper layout type.
- **Site Editor Theming**: Removed hardcoded dark background styles from the page shell and sidebar. The chrome now injects CSS variables tied to the active admin color scheme, aligning editor UI with the rest of the WordPress dashboard.
- **DataViews Extensibility**: Extracts entity view configuration out of the REST controller into a reusable function, exposing it to WordPress filters so plugins can alter `entities` view states without overriding core controllers.
- **Client-Side Media Processing**: Upload queue pauses when offline and resumes automatically. UltraHDR detection identifies JPEG uploads on ingestion, retaining originals and applying ISO 21496-1 gain maps to resized sub-sizes.

## Contribution

Released June 17, 2026. The release was driven by a broad contributor effort focused on editor stability (Real-time Collaboration reliability), design-system refinements, and preparatory work for React 19 adoption. Key contributors include @aduth, @westonruter, @desrosj, @jasmussen, and others who iterated on the React 19 compatibility layer, DataViews filtering API, block transform targeting, and media editor UI updates.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
