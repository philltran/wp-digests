# #64158: Fix preg_match null deprecation in WC_Eval_Math

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @bruberries
- **Labels:** `plugin: woocommerce`
- **Merged:** [`2982e6c`](https://github.com/woocommerce/woocommerce/commit/2982e6c0225a8e1cf1e2d851f23ffbeafbd1ac0a)
- **Discussion:** [#64158](https://github.com/woocommerce/woocommerce/pull/64158) · 7 comments · 0 reactions

## Summary

This update resolves a PHP 8.1+ deprecation notice in WooCommerce’s `WC_Eval_Math` class when evaluating mathematical shipping formulas with insufficient stack depth. Specifically, the internal stack’s `last()` method returns `null` when accessing indices beyond available elements, which was passed directly to `preg_match()`. The patch applies a null coalescing operator to safely convert `null` into an empty string before regex evaluation, eliminating the deprecation warning without altering parsing behavior.

## Impact

- **Plugin & Theme Developers**: No behavioral changes or API modifications; existing math-based shipping formulas will function identically.
- **Hosting & Platform Teams / PHP 8.1+ Environments**: Deprecation notices regarding `preg_match` receiving `null` will cease in `debug.log` when calculating formula-based costs.
- **Action Required**: No code changes needed. Update to WooCommerce 10.9 to apply the fix.

## Technical details

- **File**: `plugins/woocommerce/includes/libraries/class-wc-eval-math.php`
- **Method**: `WC_Eval_Math::nfx( $expr )`
- **Change**: Two `preg_match()` calls previously passed `$stack->last( 2 )` directly. The unified diff replaces this with `$stack->last( 2 ) ?? ''`.
- **Behavioral Impact**: When the stack depth is insufficient, `last()` returns `null`. Coalescing to an empty string ensures `preg_match()` receives a valid string parameter; evaluating against `''` yields `0`, preserving the original control flow while satisfying PHP’s type expectations for PHP 8.1+.
- A changelog file (`changelog/fix-eval-math-preg-match-null-deprecation`) was added to document the patch.

## Contribution

Opened by @bruberries and merged after review from @zhongruige and @Luc45. The author noted an upcoming leave of absence, which accelerated the review cycle. The change targets the WooCommerce 10.9 release and introduces a straightforward null-safety adjustment with no contested alternatives or design debates documented in the thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
