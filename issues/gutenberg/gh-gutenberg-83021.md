# #83021: Details: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Details`
- **Merged:** [`0c060be`](https://github.com/WordPress/gutenberg/commit/0c060be71f1a3dc2a9b5d61a5725222da5c4b8c5)
- **Discussion:** [#83021](https://github.com/WordPress/gutenberg/pull/83021) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds `shadow` support to the Details block, closing a gap where the block already supported color, border, spacing, and typography but had no shadow control. The change is a single `"shadow": true` entry in the block's `block.json` supports object, plus regenerated documentation. Because Details is a static block, the shadow CSS is serialized onto the `<details>` wrapper by `useBlockProps.save()` with no PHP template change required. This is part of the broader design-tools consistency effort tracked in #43241.

## Impact

- **Site owners / editors:** Can now apply a shadow to Details blocks via Global Styles (Blocks → Details → Shadow) or per-block in the block inspector. Block-level shadow overrides the Global Styles value. No action required on existing sites; the block renders identically until a shadow is explicitly set.
- **Theme & plugin developers:** No code changes needed. The new support is handled entirely by the block editor's existing shadow infrastructure. If you build custom Details-block patterns or templates, the `<details>` wrapper will now carry a `--wp--shadow` custom property when a shadow is applied.
- **No breaking changes, deprecations, or removed APIs.**

## Technical details

The functional change is one line in `packages/block-library/src/details/block.json`, adding `"shadow": true` to the `supports` object:

```json
"supports": {
  "align": ["full", "wide"],
  "allowedBlocks": { "__unstablePreserveBlocks": true, "style": true },
  "html": false,
  "shadow": true,
  "spacing": { "margin": true, "padding": true, "blockGap": true },
  "typography": { "fontSize": true, "lineHeight": true }
}
```

Because the Details block is static (no dynamic render callback), the shadow CSS variable is emitted onto the `<details>` wrapper element by `useBlockProps.save()` at serialization time. No change to `render.php` or any PHP file is needed.

The remaining two files in the diff are regenerated documentation:
- `docs/reference-guides/core-blocks/README.md` — `shadow` added to the Details block's supports list.
- `packages/block-library/src/details/README.md` — a new `- [shadow](…): true` line added to the supports section.

## Contribution

Authored by @aaronrobertshaw and implemented by a Claude Code agent from a predefined task. @yogeshbhutkar ran a structured test pass covering Global Styles application, per-block override, extended content (images), and custom shadows, posting a video. @talldan suggested additional responsive-viewport shadow tests (setting different shadows at Tablet/Mobile breakpoints and verifying unset behavior). The author noted that this batch of AI-assisted design-tools PRs required more detailed review comments before merging could proceed, and kept the PR open pending that.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
