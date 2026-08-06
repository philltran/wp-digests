# Security: Update `composer/ca-bundle` to version `1.5.13`.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jonathan Desrosiers
- **Committed:** 2026-07-29
- **Commit:** [`54ea90b6ce`](https://github.com/WordPress/wordpress-develop/commit/54ea90b6cee0154da7897ef50f06d4f546d05d3b)
- **Usefulness:** 3/5

## Summary

WordPress core updated the bundled `composer/ca-bundle` dependency from `1.5.12` to `1.5.13`, synchronizing the included CA certificate store with Mozilla’s latest root certificates. This update removes two deprecated root certificates (`Entrust Root Certification Authority` and `SecureSign Root CA12`) to maintain accurate TLS validation for outbound HTTP requests and address upstream security advisories.

## Impact

- **Plugin & theme developers**: No action required for standard integrations. Outbound HTTPS requests via `wp_remote_get()` or `wp_safe_remote_request()` will continue validating against the updated store.
- **Hosting & platform teams**: No configuration changes needed; the updated bundle ships automatically with core.
- **Internal PKI / legacy integrations**: Sites that explicitly trust the removed `Entrust` or `SecureSign` roots for internal services or legacy TLS handshakes may experience connection failures until those certificates are re-added or replaced.

## Technical details

The diff updates `composer.json` (under `require-dev`) to require `composer/ca-bundle` `1.5.13` and regenerates `src/wp-includes/certificates/ca-bundle.crt`. The regenerated file updates the Mozilla sync timestamp to `Thu Jul 16 03:12:01 2026 GMT` and recalculates the PEM bundle SHA256 hash. Two certificate blocks are stripped: `Entrust Root Certification Authority` and `SecureSign Root CA12`. WordPress loads this file in `WP_Http_Curl::request()` and passes it to `curl_setopt( $ch, CURLOPT_CAINFO, ... )` when cURL is the active transport, ensuring all core HTTP API calls validate against the current Mozilla root store.

## Contribution

The record carries no public discussion detail; this is a routine security maintenance commit authored by Jonathan Desrosiers and merged on 2026-07-29 to address Trac ticket #64969.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
