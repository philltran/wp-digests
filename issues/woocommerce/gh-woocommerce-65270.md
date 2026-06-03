# #65270: Use first name for greeting if available, instead of user login, on new customer account and reset password emails

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @webdados
- **Labels:** `needs: documentation`, `plugin: woocommerce`, `type: community contribution`
- **Merged:** [`587e995`](https://github.com/woocommerce/woocommerce/commit/587e995ae6fac3f3f4dda18ac7d6297966585e97)
- **Discussion:** [#65270](https://github.com/woocommerce/woocommerce/pull/65270) · 7 comments · 1 reactions

## Summary

The "new customer account" and "reset password" transactional emails now greet customers by first name instead of their login (username). Both `WC_Email_Customer_New_Account` and `WC_Email_Customer_Reset_Password` compute a `user_display_name` during `trigger()` using a three-step fallback: `WC_Customer::get_billing_first_name()` → WordPress `first_name` user meta → `user_login`. The resolved value is passed into HTML, plain-text, and block-editor template contexts. As a side effect, `WC_Email_Customer_Reset_Password::trigger()` gained an early-return guard when `get_user_by()` returns `false`, eliminating two long-standing PHPStan property-on-non-object suppressions.

## Impact

**Plugin & theme developers / stores with customised email templates:**
- **Action required if you have overridden any of these four templates:**
  - `templates/emails/customer-new-account.php` (bumped to `10.9.0`)
  - `templates/emails/customer-reset-password.php` (bumped to `10.9.0`)
  - `templates/emails/plain/customer-new-account.php` (bumped to `10.9.0`)
  - `templates/emails/plain/customer-reset-password.php` (bumped to `10.9.0`)
  - WooCommerce's template-outdated notice will flag these. Update your overrides to use `$user_display_name` instead of `$user_login` in the greeting line, or continue using `$user_login` if you prefer the old behaviour.
- **`$user_display_name` is now available** as a template variable in both email template contexts (HTML, plain, block editor). `$user_login` is still passed and unchanged.
- **No action required** for stores using unmodified WooCommerce templates.

**Hosting & platform:**
- No configuration changes. The new `WC_Customer` instantiation inside `trigger()` adds one extra object construction per email send; negligible for typical volumes.

## Technical details

**`WC_Email_Customer_New_Account` (`includes/emails/class-wc-email-customer-new-account.php`):**

Adds a `public string $user_display_name` property. Inside `trigger()`, after the `WP_User` object is loaded:

```php
$customer                = new WC_Customer( $user_id );
$first_name              = ! empty( $customer->get_billing_first_name() )
    ? $customer->get_billing_first_name()
    : $this->object->first_name;          // WP user meta
$this->user_display_name = ! empty( $first_name ) ? $first_name : $this->user_login;
```

`user_display_name` is then injected into the `wc_get_template_html()` context arrays for `get_content_html()`, `get_content_plain()`, and `get_block_editor_email_template_content()`.

**`WC_Email_Customer_Reset_Password` (`includes/emails/class-wc-email-customer-reset-password.php`):**

Same `$user_display_name` property and identical resolution logic. Additionally, `trigger()` now guards against a falsy return from `get_user_by( 'login', $user_login )`:

```php
$this->object = get_user_by( 'login', $user_login );
if ( ! $this->object ) {
    $this->restore_locale();
    return;
}
```

This removes two `WP_User|false` PHPStan suppressions from `phpstan-baseline.neon` and adds one for the now-untyped return of `trigger()`.

**Template changes (before → after):**

```php
// Before
/* translators: %s: Customer username */
<p><?php printf( esc_html__( 'Hi %s,', 'woocommerce' ), esc_html( $user_login ) ); ?></p>

// After
/* translators: %s: Customer first name, or username if name is not available */
<p><?php printf( esc_html__( 'Hi %s,', 'woocommerce' ), esc_html( $user_display_name ) ); ?></p>
```

Applied identically in all four templates (HTML and plain variants for both emails). Template version headers bumped from `10.4.0`/`10.0.0`/`9.8.0` to `10.9.0`.

## Contribution

Opened by community contributor **@webdados** (referencing issue #65269). The initial submission triggered PHPStan failures on pre-existing `WP_User|false` property accesses in `WC_Email_Customer_Reset_Password`; @webdados argued these were out of scope but ultimately submitted a fix adding the `! $this->object` guard. **@jorgeatorres** (WooCommerce core) guided regeneration of the PHPStan baseline via `pnpm --filter=@woocommerce/plugin-woocommerce phpstan:baseline` and assisted with the final fix. The PR was merged at commit `587e995`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
