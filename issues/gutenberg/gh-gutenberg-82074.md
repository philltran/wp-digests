# #82074: Site title: Add fit-text support.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Site Title`
- **Merged:** [`9b273c1`](https://github.com/WordPress/gutenberg/commit/9b273c1ff8d5d6e382b965884c607fee0a19262b)
- **Discussion:** [#82074](https://github.com/WordPress/gutenberg/pull/82074) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Site Title block (`core/site-title`) now opts into the `typography.fitText` block support, so its text can be scaled to fill the width of its container. It is exposed as a toggle in the block's Typography panel ellipsis menu, matching what Heading and Paragraph already offer. The PR also adjusts the server-side typography support so the rendered markup gets a `has-fit-text` class, and adds an editor style fix.

## Impact

**Site owners / theme authors**
- Site Title can now render full-width, container-fitted text (e.g. large hero-style site names) via the Typography panel, with no custom CSS or JS.
- Opt-in only: existing Site Title blocks are unchanged unless Fit text is toggled on.

**Plugin & theme developers**
- `core/site-title` now declares `supports.typography.fitText: true`. Code that inspects block supports for this block will see the new key.
- Frontend markup for blocks using fit text gains a `has-fit-text` class on the root element. Custom CSS targeting this class will now match on Site Title too.
- The change is marked for backport to core (7.2 changelog entry, wordpress-develop PR 13287), so the server-side rendering change will need to land in core as well.

No action required for existing sites.

## Technical details

The diff makes four functional changes:

- `packages/block-library/src/site-title/block.json`: adds `"fitText": true` to `supports.typography`.
- `lib/block-supports/typography.php`: in `gutenberg_render_typography_support()`, inside the fit-text branch that uses `WP_HTML_Tag_Processor`, adds `$processor->add_class( 'has-fit-text' );` on the first tag, before the existing `data-wp-interactive` handling.
- `packages/block-library/src/site-title/editor.scss`: adds an override so the editor doesn't break fit-text sizing:

```scss
.wp-block-site-title.has-fit-text > * {
	white-space: inherit !important;
}
```
  This counters the inline `white-space: pre-wrap` that the rich text editable sets.
- Docs and housekeeping: the core-blocks `README.md` supports line now lists `fitText`, the block's `README.md` gains a `fitText: true` entry, plus a `block-library` CHANGELOG entry and `backport-changelog/7.2/13287.md`.

The diff is limited to the `block.json` opt-in, the PHP class addition and the editor stylesheet. The fit-text logic itself (the interactivity script and controls) already existed for other blocks.

## Contribution

Opened by @jasmussen to close #82061. @t-hamano reviewed it, agreed the feature made sense, and prepared a core backport PR while making several changes to the approach. Jasmussen rebased to resolve a changelog conflict, and t-hamano confirmed it was ready to merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
