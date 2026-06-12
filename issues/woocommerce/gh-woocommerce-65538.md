# #65538: Remove 'add-to-wishlist-button' from Add to Cart + Options template parts

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @Aljullu
- **Labels:** `needs: documentation`, `plugin: woocommerce`
- **Merged:** [`dc381fd`](https://github.com/woocommerce/woocommerce/commit/dc381fd167b1fa4b5a65b137c5ca847f04def22b)
- **Discussion:** [#65538](https://github.com/woocommerce/woocommerce/pull/65538) · 7 comments · 0 reactions

## Summary

This PR removes the `woocommerce/add-to-wishlist-button` block from WooCommerce's default Add to Cart + Options template parts. The block was inadvertently included in a prior release while its underlying functionality remained behind an experimental feature flag, causing the site editor to render it as an unsupported/ignored element. Removing it resolves editor warnings and aligns the default templates with the current opt-in state of the feature.

## Impact

- **Plugin & Theme Developers**: The default Add to Cart + Options (ATCWO) template parts no longer ship with the wishlist button block. Any theme or plugin extending these templates will need to manually insert the `woocommerce/add-to-wishlist-button` block if wished-list functionality is desired.
- **Site Editors / Merchants**: Resolves invalid/ignored block warnings in the pattern editor when viewing product templates. No action required for stores that don't use wishlists.
- **Headless & REST Consumers**: Unaffected; this change only modifies static template/part HTML files and does not alter the REST API or client-facing data schema.

## Technical details

The unified diff removes `<!-- wp:woocommerce/add-to-wishlist-button /-->` from four template files under `plugins/woocommerce/templates/parts/`:
- `external-product-add-to-cart-with-options.html`
- `grouped-product-add-to-cart-with-options.html`
- `simple-product-add-to-cart-with-options.html`
- `variable-product-add-to-cart-with-options.html`

The removal is explicit in the HTML; no PHP or JavaScript code was altered. The change was necessary because the block remains behind a feature flag, and attempts to dynamically inject it via Block Hooks previously caused the DOM to silently drop the block (`{"ignoredHookedBlocks":["woocommerce\/add-to-wishlist-button"]}`). Until a dedicated product context can reliably attach the block, static inclusion was reverted. A patch-level changelog entry was added alongside the template updates.

## Contribution

Opened by @Aljullu to address a regression introduced in PR #65263, which had shipped the wishlist block before the feature flag was stabilized. Reviewed and merged by @ralucaStan and @jorgeatorres, with an immediate cherry-pick to `release/10.9`. During review, the team considered automating opt-in via Block Hooks or a wp-admin toggle, but discarded both approaches: hook-based injection failed in testing (the block was ignored in the DOM), and automatic changes were deemed disruptive for stores that do not need wishlist functionality. The final decision settled on explicit opt-in via template editing.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
