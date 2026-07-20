# #77185: Autocomplete: Skip stale triggers from completed mentions

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Feature] UI Components`, `[Package] Components`
- **Merged:** [`67ad6da`](https://github.com/WordPress/gutenberg/commit/67ad6dac1440fc45ae83329a37e9157b5aa13815)
- **Discussion:** [#77185](https://github.com/WordPress/gutenberg/pull/77185) · 7 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug in the block editor’s autocomplete system where the mention popover would immediately re-open after a user selected an inline completion (e.g., `@username`). The trigger prefix remained in the text, causing the matching logic to re-scan and re-trigger the popover on every subsequent text change. This change introduces state guards to suppress stale re-matching until the user explicitly types a new trigger.

## Impact

- **Block editor users & content authors:** The mention autocomplete popover no longer flickers or re-opens immediately after selecting a mention.
- **Plugin & theme developers:** `getAutocompleteMatch` now accepts an options object instead of positional parameters. Custom completers that directly invoke this function will need to update their call signature.
- **No action required** for standard editor usage or completers that do not directly call `getAutocompleteMatch`.

## Technical details

The diff modifies the autocomplete effect in the `@wordpress/components` package to prevent stale re-matching. It introduces two ref-based guards: `lastCompletionRef` stores the completer name and post-trigger text after an insertion, allowing `getAutocompleteMatch` to suppress re-matching when the filter value matches the completed text; and `prevRecordTextRef` tracks `record.text` across renders to skip activation during cursor-only movements. The diff also refactors `getAutocompleteMatch` to accept a single options object instead of positional parameters and simplifies control flow in `select`. A `test.fixme` marks the known edge case where multiple mentions or editing near a completed mention can still re-trigger the popover.

## Contribution

Opened and merged by @Mamaduka, with co-authors @ciampo, @mirka, @shrivastavanolo, and @chad1008. The PR closed #42925 and superseded #71373. Review feedback from @ciampo informed the guard implementation and the signature refactor for `getAutocompleteMatch`. The author noted that while this resolves the immediate re-trigger bug, a more comprehensive solution for multiple mentions or adjacent edits remains a follow-up (#77007).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
