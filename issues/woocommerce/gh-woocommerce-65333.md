# #65333: Deprecate Product Editor v2

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @gigitux
- **Labels:** `needs: documentation`, `plugin: woocommerce`, `developer advisory`
- **Merged:** [`56202f4`](https://github.com/woocommerce/woocommerce/commit/56202f48429b020b826a149b23c289663a5a21ad)
- **Discussion:** [#65333](https://github.com/woocommerce/woocommerce/pull/65333) · 4 comments · 0 reactions

## Summary

This PR formally deprecates WooCommerce's Product Editor v2 (the React/block-based product editor, internally the `product_block_editor` feature) and its entire extensibility surface, ahead of removal in **WooCommerce 11.0**. It adds runtime deprecation notices across both the JavaScript (`@woocommerce/product-editor`) and PHP layers, marks the public template/block-registration APIs and filter/action hooks as deprecated since **10.9.0**, and surfaces an in-editor warning banner that lets merchants switch back to the classic product editor in one click. The legacy `FeedbackBar` CES component is also removed as part of the cleanup. No functionality is removed yet — this release is the advisory window before the editor and its APIs are deleted.

## Impact

**Plugin & extension developers (primary audience)**
- Any extension that builds on the new product editor is on a removal clock: the editor and all its extension APIs are slated for deletion in **WooCommerce 11.0**.
- The following now emit deprecation notices when used (deprecated since 10.9.0):
  - JS: importing/using the `@woocommerce/product-editor` package, `initBlock()`, and `registerProductEditorBlockType()`.
  - PHP function: `BlockRegistry::register_block_type_from_metadata()`.
  - PHP filters: `experimental_woocommerce_product_editor_product_template_id_for_product`, `woocommerce_product_editor_product_templates`, and `woocommerce_product_form_templates`.
  - PHP block-template action hooks dispatched from `AbstractBlock`/`BlockContainerTrait` (add/remove hide conditions and block-template after-add/remove hooks).
  - `@deprecated 10.9.0` docblocks added to `ProductTemplate`, `BlockTemplateUtils`, `RedirectionController`, `Tracks`, and the `GroupInterface`/`SectionInterface`/`SubsectionInterface`/`ProductFormTemplateInterface` interfaces.
- **Action required:** plan migration of product-editor extensions to the classic editor's meta-box/`product_data` tab APIs before 11.0; expect `wc_deprecated_function`/`wc_deprecated_hook`/`wc_doing_it_wrong`-style notices in logs once you upgrade.

**Site owners / merchants**
- If the new product editor is enabled, a warning banner now appears in the editor recommending a switch to the classic editor, with a **Switch to classic editor** button that disables the feature and redirects.
- No data changes — products are unaffected by switching editors.

**Everyone else**
- No action required if you do not use or extend the new product editor.

## Technical details

**JavaScript / UI (`packages/js/product-editor`)**
- `index.ts`, `utils/init-block.ts`, and `utils/register-product-editor-block-type.ts` now register `@wordpress/deprecated` notices (version `10.9.0`, hinting removal in WooCommerce 11.0) on package export, `initBlock()`, and `registerProductEditorBlockType()`.
- `components/block-editor/block-editor.tsx` renders a `warning` `Notice` titled "Switch to the classic editor before WooCommerce 11.0" with a `Switch to classic editor` `Button`. The handler reads the `_feature_nonce` from the `admin` settings, then:

```ts
const PRODUCT_BLOCK_EDITOR_FEATURE_OPTION =
  'woocommerce_feature_product_block_editor_enabled';

await apiFetch( {
  path: `/wc/v3/settings/advanced/${ PRODUCT_BLOCK_EDITOR_FEATURE_OPTION }`,
  method: 'POST',
  data: { value: 'no' },
} );
window.location.href = classicProductEditorUrl; // fallback: disable URL with product_block_editor=0 & _feature_nonce
```

- `getClassicProductEditorUrl()` resolves the classic edit screen, using `parent_id` for `product_variation` post types (`post.php?post=…&action=edit`), falling back to `post-new.php?post_type=product`.
- The `FeedbackBar` component (`feedback-bar.tsx`, its `index.ts`, styles, test, and barrel export) is deleted, and its render is removed from `client/admin/client/products/product-page.tsx`; footer rendering is gated behind `false`.

**PHP**
- `BlockRegistry.php` adds a `DEPRECATED_SINCE` constant and replaces the public `register_block_type_from_metadata()` with a wrapper that calls `wc_deprecated_function()` then delegates to a new private `register_block_type_from_metadata_without_deprecation_notice()`; internal `register_block()` calls the private helper to avoid self-triggering the notice.

```php
// Before: public method did the registration directly.
// After:
public function register_block_type_from_metadata( ... ) {
    wc_deprecated_function( __METHOD__, self::DEPRECATED_SINCE );
    return $this->register_block_type_from_metadata_without_deprecation_notice( ... );
}
```

- `Init.php` and `ProductFormsController.php` add `DEPRECATED_SINCE` constants and emit `wc_deprecated_hook()` (guarded by `has_filter()`) for the product-template/product-form filters.
- `AbstractBlock.php` and `BlockContainerTrait.php` swap `do_action()` for `wc_do_deprecated_action()`, computing the area-specific `$hook_name` once and passing it through to the error handlers.
- Tests in `class-wc-rest-layout-templates-controller-tests.php` and `BlockTemplateTest.php` add `setExpectedDeprecated()` calls for the now-deprecated block-template hooks.

No DB schema or REST schema changes; the only REST interaction is a POST to the existing `wc/v3/settings/advanced` option route to flip the feature off.

## Contribution

Opened and authored by **@gigitux** as a `developer advisory`, closing issue #65318, and merged in commit `56202f4` with the milestone auto-assigned to the next WooCommerce version. Reviewers flagged for testing were **@jamesckemp**, **@louwie17**, and **@dinhtungdu**. The change is mechanical and uncontroversial — CodeRabbit's automated review generated no actionable comments, and the only pre-merge warning was a docstring-coverage threshold miss (79.17% vs 80%). The discussion contains no notable design debate; the scope (deprecation notices + classic-editor switch banner, plus opportunistic removal of the `FeedbackBar`) was confirmed in-scope against the linked issue.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
