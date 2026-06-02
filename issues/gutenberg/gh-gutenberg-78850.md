# #78850: Dashboard: Fix Add widget error on non-secure HTTP origins

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`
- **Merged:** [`133de8d`](https://github.com/WordPress/gutenberg/commit/133de8dd2fd651d292af2caf0134bee9bb90f21b)
- **Discussion:** [#78850](https://github.com/WordPress/gutenberg/pull/78850) · 3 comments · 1 reactions

## Summary

The Dashboard inserter's **Add widget** action threw a `TypeError` on plain `http://` origins because `createDashboardWidget()` used `crypto.randomUUID()` to stamp new widget instance IDs. `crypto.randomUUID()` is restricted to [secure contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Secure_Contexts) (HTTPS or `localhost`), causing a hard failure for anyone running WordPress over insecure HTTP — a common local-dev scenario. The fix replaces that call with the `uuid` npm package (`v4`), which has no context restriction. The UUID v4 output format is identical, so no persisted widget data is affected.

## Impact

**Plugin & theme developers / Dashboard integrators**
- No API surface change. `createDashboardWidget()` continues to return the same `DashboardWidget` shape with a UUID v4 instance ID.
- No migration or code changes required.

**Local / staging environments on plain HTTP**
- The `TypeError` thrown when clicking **Add widget** on insecure origins is resolved. Sites proxied over `http://` (e.g., tunnels, non-localhost dev setups) can now use the widget inserter.

**HTTPS / localhost environments**
- No behavioral change; widget insertion already worked.

**No breaking changes or deprecations.**

## Technical details

The affected file is `routes/dashboard/widget-dashboard/create-dashboard-widget.ts`. `createDashboardWidget()` previously generated its widget instance ID with the native Web Crypto API:

```ts
// before
const id = crypto.randomUUID();
```

The fix imports `v4` from the `uuid` package and uses it instead:

```ts
// after
import { v4 as uuid } from 'uuid';
// …
const id = uuid();
```

`uuid ^14.0.0` is declared as a new entry in `dependencies` inside `routes/dashboard/package.json`, following the same pattern used by other packages in the Gutenberg monorepo. `package-lock.json` is updated accordingly. The size bot confirmed the bundled output is unchanged (0 B delta across all tracked build artifacts), consistent with `uuid` already being present in the dependency graph.

## Contribution

Opened and merged by @jameskoster (commit `133de8d`). The PR attracted minimal discussion — a single automated comment from `github-actions[bot]` confirming zero bundle-size impact, plus one reaction. No alternative approaches were discussed in the thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
