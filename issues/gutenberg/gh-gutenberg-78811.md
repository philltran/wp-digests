# #78811: RTC: Prevent slower polling filters

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shekharnwagh
- **Labels:** `[Type] Enhancement`, `[Feature] Real-time Collaboration`, `[Package] Sync`
- **Merged:** [`4822ef8`](https://github.com/WordPress/gutenberg/commit/4822ef807880d1c1efdcc3172aeb499b899f0b74)
- **Discussion:** [#78811](https://github.com/WordPress/gutenberg/pull/78811) · 3 comments · 0 reactions

## Summary

This change hardens the `@wordpress/sync` HTTP polling provider used by Real-time Collaboration (RTC) so that the polling-interval filters can only make active polling *faster*, never slower. Previously, hosts could use these filters to raise the polling interval high enough that collaboration felt laggy or broken. The filtered values are now passed through a small helper that accepts only positive finite numbers, allows values below the defaults, and clamps anything slower back to the defaults (`4000ms` when solo, `1000ms` with collaborators). Invalid values fall back to the defaults.

## Impact

**Hosting & platform teams**
- The RTC polling-interval filters in `@wordpress/sync` are now one-directional: you can opt into *more aggressive* (faster) polling, but you can no longer slow polling below the built-in defaults. Any existing filter returning a value larger than the default (e.g. `10000`) is ignored and clamped to the default.
- Non-finite or non-positive filter return values are rejected and fall back to the defaults.

**Plugin & theme developers / collaboration integrators**
- If you relied on these filters to reduce RTC request volume by increasing the interval, that lever no longer works — slower values are clamped. Faster values continue to apply.

**Site owners / end users**
- No action required. The default collaboration experience (`4000ms` solo / `1000ms` with collaborators against `/wp-sync/v1/updates`) is now protected from being degraded by host configuration.

No APIs were removed; the filters still exist but their accepted range is narrowed to faster-than-default values.

## Technical details

The change lives in the `@wordpress/sync` package, in `packages/sync/src/providers/http-polling/config.ts`, with coverage in `packages/sync/src/providers/http-polling/test/config.test.ts`.

The active polling-interval filters are now routed through a helper that:
- accepts only **positive, finite** numbers;
- permits values **faster** than the default (smaller interval) and uses them;
- **clamps slower values to the default**;
- falls back to the default for invalid input.

Defaults are `4000ms` when editing solo and `1000ms` when collaborators are present, driving polling of the `/wp-sync/v1/updates` REST route.

Behavioral before/after for a filter that returns a larger interval:

```js
// Filter returns 10000 (slower than the 4000ms solo default)
// Before: polling interval becomes 10000ms (degraded collaboration)
// After:  value is clamped back to 4000ms

// Filter returns 1000 (faster than the 4000ms solo default)
// Before & After: 1000ms is honored
```

New unit tests cover the default, faster, slower, and invalid filtered-value cases. The sync docs were updated to reflect the current `4000ms`/`1000ms` defaults and the faster-only filter semantics. Build size impact is negligible (`build/scripts/sync/index.min.js` +48 B).

## Contribution

Authored by **@shekharnwagh** and merged as commit `4822ef8`, closing issue #77500. The PR is labeled `[Type] Enhancement`, `[Feature] Real-time Collaboration`, and `[Package] Sync`. Discussion was thin — the only substantive automated note was the bundle-size bot reporting a +48 B change — so there is no recorded debate over rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
