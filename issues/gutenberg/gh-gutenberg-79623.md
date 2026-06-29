# #79623: Icons Registry: Allow digits and underscores in icon slugs

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `No Core Sync Required`, `[Feature] Icons`
- **Merged:** [`1b7b125`](https://github.com/WordPress/gutenberg/commit/1b7b125568d1fedd35ad589b5058215ac07483a4)
- **Discussion:** [#79623](https://github.com/WordPress/gutenberg/pull/79623) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This PR relaxes the validation rules for icon slugs in the `WP_Icons_Registry_Gutenberg` class to permit digits and underscores within slugs. Previously, only lowercase letters and hyphens were allowed. This enhancement resolves real-world constraints where popular icon identifiers (e.g., `html5`, `500px`) contain digits that previously triggered validation failures.

## Impact

- **Plugin & Theme Developers**: Developers registering icons via the experimental `WP_Icons_Registry_Gutenberg` class can now use slugs containing digits and underscores (e.g., `my_icon`, `ui-1`).
- **Breaking/Deprecation**: No breaking changes. The previous regex `/^[a-z][a-z0-9-]*$/` is replaced, but the change expands allowed characters rather than restricting them.
- **Action Required**: None for most users; those relying on previously rejected slugs will now see them accepted. Ensure `WP_Icons_Registry_Gutenberg` implementations handle the updated validation gracefully.

## Technical details

The diff modifies `lib/class-wp-icons-registry-gutenberg.php`. The validation regex in `WP_Icons_Registry_Gutenberg::register()` was updated:
```php
// Before
if ( ! preg_match( '/^[a-z][a-z0-9-]*$/', $unqualified_name ) ) {

// After
if ( ! preg_match( '/^[a-z0-9]([a-z0-9_-]*[a-z0-9])?$/', $unqualified_name ) ) {
```
This enforces that slugs must start and end with a lowercase letter or digit, while permitting hyphens and underscores anywhere in the middle. The associated `_doing_it_wrong()` message was updated to reflect the new constraints. PHPUnit tests in `phpunit/experimental/class-wp-icons-registry-gutenberg-test.php` were refactored into a data provider to validate digits at various positions and underscore placement, while explicitly testing that uppercase characters and leading/trailing underscores remain invalid.

## Contribution

Opened and merged by @t-hamano with co-authorship from @mcsf as a follow-up to #77260. The PR focuses strictly on relaxing the validation regex and updating the corresponding unit tests. Labeled `No Core Sync Required`, indicating it targets the Gutenberg plugin's isolated implementation for now. Merged in commit `1b7b125`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
