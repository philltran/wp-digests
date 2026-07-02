# #79568: Style Engine: Preserve important gradient declarations

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Package] Style Engine`, `[Feature] Style States`
- **Merged:** [`690d950`](https://github.com/WordPress/gutenberg/commit/690d95098454469aa758d260cf4160eea288bb99)
- **Discussion:** [#79568](https://github.com/WordPress/gutenberg/pull/79568) · 3 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

Fixes a regression where responsive background gradients vanished on the frontend when block state styles required `!important`. The Style Engine previously appended `!important` inline before running CSS sanitization, causing gradient functions to be rejected by `safecss_filter_attr()`. Declarations now track importance as a structured option, sanitize first, and conditionally append `!important` afterward.

## Impact

- **Block & theme developers**: Responsive state gradients now render correctly on the frontend. No immediate code changes required unless you manually inject importance flags into Style Engine declarations.
- **Developers using `WP_Style_Engine_CSS_Declarations`**: The signature of `add_declaration()` now accepts an optional `$options` array. Inline `!important` strings in values are no longer processed by the engine's sanitizer.
- **Site owners & editors**: No action required; responsive states now apply importance safely after sanitization.

## Technical details

The core change reworks how the Style Engine handles declaration metadata:
- `WP_Style_Engine_CSS_Declarations` now maintains a `$declaration_options` map. `add_declaration()` accepts an optional third argument `array $options = array()`, and `filter_declaration()` runs `safecss_filter_attr()` first, appending ` !important` only if the sanitized value is non-empty and contains no semicolons.
- In `lib/block-supports/states.php`, state declarations are no longer suffixed with inline `!important`. Instead, they are instantiated as `WP_Style_Engine_CSS_Declarations_Gutenberg` objects and passed to the engine with `array( 'important' => true )` in the options array.
- The processor's rule identity JSON now includes declaration options to prevent unintended CSS merging across different importance states.

**Before/After pattern:**
```php
// Before (inline string, stripped by sanitization)
$declarations['background-image'] = 'linear-gradient(...) !important';

// After (structured option, appended post-sanitization)
$decls = new WP_Style_Engine_CSS_Declarations_Gutenberg();
$decls->add_declaration( 'background-image', 'linear-gradient(...)', array( 'important' => true ) );
```

## Contribution

Opened and merged by @ramonjd with co-authorship from @andrewserong (merge commit `690d950`). The PR addressed an architectural mismatch where inline `!important` strings conflicted with CSS sanitization during state rendering. Review focused on ensuring the new options structure preserved rule identity for merging and didn't break existing block support test expectations. Merged following successful PHP and JS test suites.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
