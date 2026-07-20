# #80061: design-system-mcp: Use fixed version for alpha MCP server

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aduth
- **Labels:** `[Type] Bug`, `[Package] Design System MCP`
- **Merged:** [`1711dda`](https://github.com/WordPress/gutenberg/commit/1711dda8d6adf3f3b5a80278db8dc565a8f34b39)
- **Discussion:** [#80061](https://github.com/WordPress/gutenberg/pull/80061) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/design-system-mcp` package now pins its `@modelcontextprotocol/server` dependency to an exact version (`2.0.0-alpha.2`) instead of a caret range. This prevents fresh installs from automatically resolving to newer prerelease versions that introduced breaking API changes, which previously caused the MCP server to fail on startup with a `SyntaxError` when importing `StdioServerTransport`.

## Impact

- **MCP consumers & developers**: No code changes required, but the package will no longer auto-update to newer `@modelcontextprotocol/server` prereleases that break the server.
- **Gutenberg core contributors**: The `syncpack` semver enforcement rule now explicitly exempts `@modelcontextprotocol/server` to allow pinned prerelease versions.
- **Existing installations**: No action required; the lockfile will update on the next `npm install`.

## Technical details

The diff updates `packages/design-system-mcp/package.json` to change the `@modelcontextprotocol/server` dependency from `^2.0.0-alpha.2` to `2.0.0-alpha.2`, with corresponding updates to `package-lock.json`. To satisfy the repository's `syncpack` rule that enforces uniform caret ranges, `syncpack.config.mjs` adds a `semverGroups` entry that explicitly exempts `@modelcontextprotocol/server` from the caret-range enforcement by setting `range: ''`. The upstream breakage occurred in `beta.1` of `@modelcontextprotocol/server`, which removed the named export `StdioServerTransport` that `packages/design-system-mcp/bin/design-system-mcp.mjs` relies on.

## Contribution

Opened and merged by @aduth. During review, @ciampo flagged a `syncpack` lint error because the repository enforces uniform caret ranges across dependencies. @aduth identified that `syncpack` required a manual exemption for this specific prerelease dependency and updated `syncpack.config.mjs` to allow it. @manzoorwanijk was looped in for awareness. The fix was merged in commit `1711dda`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
