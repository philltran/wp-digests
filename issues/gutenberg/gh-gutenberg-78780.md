# #78780: Block Visibility: Keep hide-everywhere working after a block opts out of visibility support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Feature] Block API`, `Backwards Compatibility`, `[Block] HTML`, `Backport to WP Minor Release`, `[Block] Shortcode`
- **Merged:** [`bca81ff`](https://github.com/WordPress/gutenberg/commit/bca81ff16c05c786d5875bc70ca7ea55f7615a7c)
- **Discussion:** [#78780](https://github.com/WordPress/gutenberg/pull/78780) · 5 comments · 0 reactions

## Summary

`gutenberg_render_block_visibility_support()` in `lib/block-supports/block-visibility.php` now suppresses blocks with `metadata.blockVisibility: false` in their serialized attributes regardless of whether the block type currently opts into `visibility` block support. The regression was introduced by #76138, which removed visibility support from `core/html` and `core/shortcode`; blocks those users had previously hidden via the visibility toggle reappeared on the front end because the support check short-circuited before the attribute was ever read. This fix reorders the guard so the "hide everywhere" state is always honoured as a render-layer suppression signal.

## Impact

**Site owners**
- Blocks hidden with the "Hide everywhere" visibility control before #76138 (Gutenberg ≥ 19.8) now correctly stay hidden. No action required.

**Plugin & theme developers**
- Any block that opts *out* of `visibility` support but has post content serialized with `"metadata":{"blockVisibility":false}` will now be suppressed on the front end. This is intentional — it preserves user intent across support-flag changes — but could be surprising if you deliberately removed the support to un-hide previously hidden blocks.
- No API, filter, or hook changes. No migration required.
- Developers who want selective support (e.g. keep "hide everywhere" but drop viewport-based hiding) may revisit once a granular support shape — `"visibility": { "viewport": false, "everywhere": true }` — is formally proposed; that approach was discussed but not implemented here.

**Hosting & platform**
- PHP-only change; no JS bundle, REST schema, or DB impact.

**Headless & REST consumers**
- No change to REST output. Suppression occurs in the server-side render path.

## Technical details

**File:** `lib/block-supports/block-visibility.php`

The function `gutenberg_render_block_visibility_support()` previously gate-checked `block_has_support( $block_type, 'visibility', true )` before ever reading `$block['attrs']['metadata']['blockVisibility']`. Removing visibility support from a block type caused the function to return `$block_content` unconditionally, ignoring a stored `false` value.

**Before:**
```php
if ( ! $block_type || ! block_has_support( $block_type, 'visibility', true ) ) {
    return $block_content;
}
$block_visibility = $block['attrs']['metadata']['blockVisibility'] ?? null;
if ( false === $block_visibility ) {
    return '';
}
```

**After:**
```php
if ( ! $block_type ) {
    return $block_content;
}
$block_visibility = $block['attrs']['metadata']['blockVisibility'] ?? null;
// Hide regardless of current support to preserve intent from when support was active.
if ( false === $block_visibility ) {
    return '';
}
if ( ! block_has_support( $block_type, 'visibility', true ) ) {
    return $block_content;
}
```

The `block_has_support()` guard is preserved for array-shaped `$block_visibility` values (viewport-based rules), so blocks without visibility support are still unaffected by that branch.

The corresponding PHPUnit test in `phpunit/block-supports/block-visibility-test.php` was updated: `test_block_visibility_support_shows_block_when_support_not_opted_in` → `test_block_visibility_support_hides_block_when_visibility_false_even_without_support`, with the assertion flipped from `assertSame( $block_content, $result )` to `assertSame( '', $result )`.

A backport entry at `backport-changelog/7.1/12053.md` links to `wordpress-develop` PR #12053 for the corresponding core backport.

## Contribution

@t-hamano opened the PR to address #78701 (the regression) and noted upfront uncertainty about the approach, intending to file a parallel `wordpress-develop` core PR before merging. @talldan proposed a more surgical alternative — a granular support shape (`visibility: { viewport: false, everywhere: true }`) that would let blocks re-expose the "hide everywhere" UI without supporting viewport rules — but that design was deferred rather than blocked on here. @ramonjd and @Mamaduka are listed as co-authors. The PR carries the `Backport to WP Minor Release` label, with `backport-changelog/7.1/12053.md` tracking the core-side PR.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
