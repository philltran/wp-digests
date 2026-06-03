# #78185: design-system-mcp: Update get_components to optionally support multiple names

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aduth
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Design System MCP`
- **Merged:** [`18e5ff7`](https://github.com/WordPress/gutenberg/commit/18e5ff71363a92ff76c48941aacbb803a59c94ea)
- **Discussion:** [#78185](https://github.com/WordPress/gutenberg/pull/78185) · 3 comments · 0 reactions

## Summary

Updates the Design System MCP `get_component_details` tool to accept either a single component name string or an array of strings in one request. This allows AI agents to fetch metadata for multiple components simultaneously, reducing round-trip latency and preventing incorrect assumptions about component APIs when constructing multi-component UI layouts.

## Impact

- **AI/Agent developers & MCP server implementers**: Can pass `string | string[]` to the tool without schema validation failures. No breaking changes; single-string inputs remain fully supported.
- **Theme/plugin developers using design system primitives**: Indirect benefit from more accurate, context-aware agent outputs when building complex layouts.
- **Platform & hosting teams**: No migration or configuration changes required. The enhancement operates entirely within the MCP tooling layer and maintains strict backward compatibility.

## Technical details

- Modifies the input schema for the `get_component_details` handler to accept `string | string[]` for the component name parameter.
- The server-side logic normalizes single-string inputs into a single-element array, then iterates over the resolved names to collect design system metadata.
- Multiple results are concatenated using a markdown horizontal rule (`---`) to maintain clear separation within the agent's context window.
- Changes apply to the `Design System MCP` package/tooling; schema validation is enforced by the underlying MCP framework before handler execution.

## Contribution

PR #78185 was authored by @aduth with co-authorship from @mirka and @ciampo. The scope was narrowly focused on optimizing agent tool-use patterns for complex component lookups, rejecting broader registry refactors in favor of a lightweight schema and output-formatting update. Merged as a backward-compatible enhancement to improve AI-assisted workflow reliability.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
