# #65247: Link email events orders store actions

- **Source:** woocommerce/woocommerce
- **Type:** Pull request
- **Author:** @jonathanbossenger
- **Labels:** `needs: documentation`, `plugin: woocommerce`, `type: community contribution`, `focus: e2e tests`
- **Merged:** [`d852f6b`](https://github.com/woocommerce/woocommerce/commit/d852f6b9dea5617e6a96eb038e5fbaa9951db860)
- **Discussion:** [#65247](https://github.com/woocommerce/woocommerce/pull/65247) · 8 comments · 0 reactions

## Summary

WooCommerce's `EmailLogger` service now records a private order note for every transactional email send or failure that is associated with a `WC_Order`. Previously the logger only wrote entries to the WooCommerce log files (`WooCommerce → Status → Logs`); now the same send/failure events surface directly on the order's admin timeline. Each note states whether the email "sent" or "failed to send," and failure notes include the captured `wp_mail_failed` error reason with any embedded email addresses redacted. This is a follow-up to the EmailLogger work introduced in #64491 and #64859.

## Impact

- **Site owners / store admins:** Each transactional email tied to an order (e.g. *Processing order*) now leaves a private order note like `Email "Processing order" sent.` or `Email "Processing order" failed to send: SMTP connect() failed.` on the order screen. Expect more notes on order timelines. Notes are private (not customer-facing).
- **Plugin & extension developers:** A new filter `woocommerce_email_log_add_order_note` lets you suppress the note while still allowing the log entry to be written. The existing `woocommerce_email_log_enabled` filter suppresses both the log entry and the note.
- **Test-suite maintainers:** Code that counts order notes via `wc_get_order_notes()` may now see an extra note per email send. The PR updates `OrderActionsRestControllerTest` and two Playwright specs accordingly — if you assert on note counts or positional note selectors, you will need similar adjustments.
- **No action required** for most installs; the behavior is additive and gated. Note: a follow-up (#65371) addresses a side effect where the email-preview "Send test" dummy order (`set_id(12345)`) could write a note to a real order #12345.

## Technical details

Change lives in `plugins/woocommerce/src/Internal/Email/EmailLogger.php`. After logging in `handle_woocommerce_email_sent()`, it now calls a new private method `maybe_add_order_note( $email->object, $email_id, $email, (bool) $success, $last_mail_error )`.

`maybe_add_order_note()`:
- Returns early unless `$email->object instanceof WC_Order` (so product/user-related emails like *New account* are skipped).
- Applies the new filter `apply_filters( 'woocommerce_email_log_add_order_note', true, $email_id, $email, $wc_object )` and bails if it returns false.
- Builds the note label from `$email->get_title()`, falling back to `$email_id`.
- Composes one of three localized strings: `Email "%s" sent.`, `Email "%1$s" failed to send: %2$s.` (with `$this->redact_emails( $error_reason )`), or `Email "%s" failed to send.` when no reason was captured.
- Persists via `$wc_object->add_order_note( $note, 0, false, array( 'note_group' => OrderNoteGroup::EMAIL_NOTIFICATION ) )` — a private note tagged with the `EMAIL_NOTIFICATION` note group (`OrderNoteGroup` from `Automattic\WooCommerce\Internal\Orders`).

The docblock types `$wc_object` as `mixed` because `$email->object` resolves to `bool|object`; narrowing happens via the `instanceof` check.

Before/after on the order timeline:

```
Before:  (email send only written to WooCommerce → Status → Logs)
After:   Email "Processing order" sent.
         Email "Processing order" failed to send: Recipient [redacted_email] refused.
```

New hook:

```php
// Suppress the order note but keep the log entry.
add_filter( 'woocommerce_email_log_add_order_note', '__return_false' );
```

No DB schema or REST changes. Tests: seven new methods in `EmailLoggerTest.php` cover success/failure notes, colon-form error reason, email redaction (`[redacted_email]`), non-order skip, and both filter suppressions; `create_mock_email()` now stubs `get_title()`. `OrderActionsRestControllerTest` adds `filter_out_email_logger_notes()` to exclude notes whose content begins with `Email "`. Playwright specs (`create-order.spec.ts`, `order-refund.spec.ts`) switch from positional `.system-note >> nth=N` / `div.note_content` selectors to content-filtered `.filter({ hasText: … })` + `toBeVisible()`.

## Contribution

Community contribution by @jonathanbossenger, a follow-up to the EmailLogger work in #64491 and #64859. @jobthomas pushed a fixup commit ([a9c5105](https://github.com/woocommerce/woocommerce/pull/65247/commits/a9c51051be)) to get CI green — relaxing the `maybe_add_order_note()` docblock to `mixed` for PHPStan, replacing a short ternary and re-aligning `@param` columns for PHPCS, adding the `filter_out_email_logger_notes()` helper so REST tests count only endpoint notes, swapping the brittle positional E2E locators, and tidying changelog files. @Konamiman approved pending the lint fixes and merged (`d852f6b`), targeting the next release (10.9). During review @mahangu's AI regression pipeline flagged that the email-preview "Send test" dummy order (created with `set_id(12345)` in `EmailPreview.php`) could write a real note to an existing order #12345 since `wp_insert_comment()` doesn't verify the post exists; @jobthomas confirmed and addressed it separately in #65371.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
