# #78953: ui/AlertDialog: Fix footer layout style override

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] UI`
- **Merged:** [`7419ca3`](https://github.com/WordPress/gutenberg/commit/7419ca39aa7ec9f49ed71720503de763273bc017)
- **Discussion:** [#78953](https://github.com/WordPress/gutenberg/pull/78953) · 9 comments · 0 reactions

## Summary

The `AlertDialog` component's footer layout previously collapsed to a row due to non-deterministic CSS stylesheet injection order. Because both the shared `.footer` class and the `.footer-column` override resided in the same CSS layer, precedence flipped when unrelated components lazy-loaded their stylesheets. The fix moves the override into the higher-priority `wp-ui-compositions` layer and uses CSS module `composes` to bind it to the base footer class, guaranteeing a consistent column layout regardless of load order.

## Impact

- **Gutenberg UI developers**: Affects consumers of the internal `packages/ui` tree. No code migration required; the `AlertDialog` component now enforces deterministic layer precedence.
- **Plugin/Theme developers**: No action required. This is an isolated fix within the `@wordpress/ui` package and does not modify PHP APIs, REST schemas, or public block.json fields.
- **Component authors**: The `packages/ui/CONTRIBUTING.md` documentation was updated to enforce that overrides extending shared primitives must live in a higher layer (typically `wp-ui-compositions`) and use `composes` to keep base and override classes bound together.

## Technical details

- **Files modified**: `packages/ui/src/alert-dialog/popup.tsx`, `packages/ui/src/alert-dialog/style.module.css`, `packages/ui/CONTRIBUTING.md`
- **CSS Layering & Composition**: The `.footer-column` rule was moved into `@layer wp-ui-compositions { ... }`. This higher layer guarantees precedence over the base `wp-ui-components` layer, eliminating non-deterministic wins based on `<style>` tag order. The rule now declares `composes: footer from "../utils/css/overlay-chrome.module.css";`, ensuring the shared chrome styles are always applied alongside the override.
- **Markup Simplification**: In `popup.tsx`, the footer element no longer uses `clsx()` to manually merge `overlayChromeStyles.footer` and `alertDialogStyles['footer-column']`. It now solely references `alertDialogStyles['footer-column']`, relying on the CSS module composition logic to handle class concatenation.
- **Before/After Patterns**:
```tsx
// Before (popup.tsx)
<div className={ clsx( overlayChromeStyles.footer, alertDialogStyles[ 'footer-column' ] ) }>

// After (popup.tsx)
<div className={ alertDialogStyles[ 'footer-column' ] }>
```
```css
/* Before (style.module.css) */
.footer-column {
  flex-direction: column;
}

/* After (style.module.css) */
@layer wp-ui-compositions {
  .footer-column {
    composes: footer from "../utils/css/overlay-chrome.module.css";
    flex-direction: column;
  }
}
```

## Contribution

Opened by `@ciampo` as a bug fix for non-deterministic CSS layering in the UI package. During review, `@aduth` suggested moving the `composes` declaration to the base styles instead of using it on the override. `@ciampo` clarified that layer precedence requires the override to explicitly reside in `wp-ui-compositions`, and both approaches were ultimately combined: the override lives in the higher layer while still binding classes via `composes`. Merged with co-authors `@aduth` and `@jameskoster` after confirming the fix resolves the regression without increasing bundle size.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
