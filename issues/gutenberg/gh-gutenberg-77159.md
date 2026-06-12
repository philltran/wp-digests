# #77159: design-system-mcp: Add new package for design system MCP tooling

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aduth
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `Design System`, `[Package] UI`
- **Merged:** [`6e78862`](https://github.com/WordPress/gutenberg/commit/6e78862085a43b41b2372c28bbada45aebdc0a1b)
- **Discussion:** [#77159](https://github.com/WordPress/gutenberg/pull/77159) · 10 comments · 0 reactions

## Summary

Introduces the new `@wordpress/design-system-mcp` package, a local Model Context Protocol (MCP) server that exposes WordPress design system tokens and component metadata directly to AI development agents. This enables developers to programmatically query design tokens and block component specifications during AI-assisted coding workflows without requiring external hosting or session management.

## Impact

- **Plugin & Theme Developers / Agency Engineers**: No breaking changes or migration required. The package is opt-in for developers incorporating AI agents into their build or development workflows.
- **AI Agent Configurations**: Requires manual configuration of the local Node binary via `mcp.json` or client-specific CLI commands to register the new MCP tools.
- **Hosting & Platform**: No server-side impact; operates entirely as a local, stateless process with no external dependencies.

## Technical details

- Adds the `packages/design-system-mcp` directory containing a Node binary at `bin/design-system-mcp.mjs`.
- Exposes three MCP tools: `get_components`, `get_component_details`, and `get_design_tokens`.
- Sources data directly from existing build artifacts: design token documentation generated via the Terrazzo plugin from `packages/theme/tokens`, and component metadata exported to JSON in the Gutenberg Storybook manifests (referenced as `components.json`).
- Replaces an initially considered implementation leveraging the WordPress Abilities API + `mcp-adapter` due to HTTP transport constraints, session management overhead, and inability to cleanly support anonymous/public access patterns.

## Contribution

Opened and merged by @aduth, closing issue #77206 and referencing related manifest work in #74626. During review, an alternative approach using the WordPress Abilities API with `mcp-adapter` was evaluated but explicitly rejected due to hosting constraints and stateful session requirements for anonymous access. The local binary approach was adopted to keep data fetching static, avoid external service dependencies, and align with common MCP server setup patterns.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
