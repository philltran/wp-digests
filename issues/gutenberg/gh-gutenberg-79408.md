# #79408: UI: Update @base-ui/react to 1.6.0

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`37e20ae`](https://github.com/WordPress/gutenberg/commit/37e20aef79343d151ce1dbd7cb0dd70d655d4ea9)
- **Discussion:** [#79408](https://github.com/WordPress/gutenberg/pull/79408) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Updates the `@base-ui/react` dependency in `@wordpress/ui` from `1.5.0` to `1.6.0`, along with its transitive dependencies `@base-ui/utils` and `reselect`. This upgrade brings upstream fixes for accessibility, focus management, positioning, validation, and interaction behavior across Base UI primitives used by Gutenberg's UI package. No local API changes or migration steps are required for consumers of `@wordpress/ui`.

## Impact

- **Plugin & theme developers:** No action required. The public API surface of `@wordpress/ui` remains unchanged, and existing wrappers continue to function without modification.
- **Gutenberg core contributors:** Existing unit and e2e tests for affected components (`Select`, `Combobox`, `Dialog`, `Drawer`, `Field`, `Popover`, `Tabs`, `Tooltip`, etc.) pass without assertion changes.
- **Hosting & platform teams:** No configuration or migration steps needed. The `package-lock.json` update is self-contained.

## Technical details

The diff modifies `packages/ui/package.json` to bump the `@base-ui/react` dependency from `^1.5.0` to `^1.6.0`. The `package-lock.json` resolution updates reflect the new versions: `@base-ui/react` `1.5.0` → `1.6.0`, `@base-ui/utils` `0.2.9` → `0.3.1`, and `reselect` `5.1.1` → `5.2.0`. A corresponding entry was added to `packages/ui/CHANGELOG.md` under `### Internal`. The update pulls in upstream fixes for focus return in `AlertDialog.Popup`, viewport positioning in `Dialog`/`Popover`/`Tooltip`, and combobox/autocomplete state management, all of which are consumed through `@wordpress/ui`'s wrapper components without requiring local code changes.

## Contribution

Authored by @ciampo and reviewed by @aduth, the PR was merged after an upstream regression was flagged during review. The author and reviewer agreed the regression was not blocking, and proceeded after upstream maintainers confirmed no patch release was planned. The change was manually verified against the Base UI v1.6.0 release notes and existing test suites before merging.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
