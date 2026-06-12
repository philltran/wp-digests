# #77439: Try: Remove Tab (Tab list item) block 

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Package] Icons`, `[Block] Tabs`
- **Merged:** [`0c36e05`](https://github.com/WordPress/gutenberg/commit/0c36e05a2382b01b7751bdad15a9c176f53f838e)
- **Discussion:** [#77439](https://github.com/WordPress/gutenberg/pull/77439) · 11 comments · 0 reactions

## Summary

Removes the `core/tab` block entirely, migrating tab button management from nested inner blocks to a declarative `tabs` attribute on the `core/tab-list` block. This shift simplifies editor styling workflows by allowing single-level CSS application to all tab buttons and prepares the component for Interactivity API–driven state management (e.g., active states). Existing content using `core/tab` will be affected by this breaking structural change.

## Impact

['- **Breaking change for experimental tabs users**: The `core/tab` block is completely removed from the ecosystem. It cannot be inserted as an inner block, and existing `core/tab-list` > `core/tab` structures will break until migrated to the new attribute model.', '- **Theme & Plugin Developers**: Code that programmatically adds/retrieves `core/tab` blocks or relies on the `allowedBlocks` relationship must be updated. Styling now targets `.wp-block-tab-list button` directly; per-instance tab styling is no longer supported in core (delegated to plugin-level extensions if needed).', "- **Migration/Configuration**: Existing markup must be adapted to read from the `core/tab-list` block's new `tabs` attribute array. No automatic migration is provided."]

## Technical details

- **Block Registration & Manifest**: `core/tab` is removed from `docs/manifest.json` and `packages/block-library/src/index.js`. The `tab-list` block no longer declares `"allowedBlocks": [ "core/tab" ]`.
- **Attribute Definition (`block.json`)**: The `tabs` attribute is added to `core/tab-list` with `"type": "array"`, `"source": "query"`, and `"selector": "button"`. A `query` schema extracts the `label` from each button's HTML. A `selectors` map targets `.wp-block-tab-list button` for border, color, and spacing variants.
- **Editor Rendering (`edit.js`)**: Replaces `useInnerBlocksProps` with a direct render loop over `context['core/tabs-list']`. Tab buttons are now hardcoded `<button>` elements rendered via `RichText`, with inline click handlers managed through Interactivity context/state. Focus management was added to the new button markup.
- **Server-Side Rendering (`index.php`)**: The render callback abandons iterating `$block->parsed_block['innerBlocks']`. Instead, it uses `WP_HTML_Tag_Processor` to loop through `<button>` tags, injecting per-item attributes (`id`, `aria-controls`) and Interactivity directives (`data-wp-on--click`, `data-wp-bind--aria-selected`, `data-wp-context`) directly onto the markup.
- **Styling**: `.wp-block-tab-list` CSS was simplified to a flex layout, removing inner block list wrappers. An active indicator overlay was added via `::before`. Editor styles were moved from a separate handle to a unified `style-index.css`.

## Contribution

Opened by @t-hamano to resolve styling friction and align the tabs component with Interactivity API patterns. Reviewed by @jasmussen and @iamtakashi, who endorsed the trade-off of removing per-tab instance styling in core in favor of simpler unified control and future-proof state management. Merged after confirming the block's experimental status permits this disruptive refactor.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
