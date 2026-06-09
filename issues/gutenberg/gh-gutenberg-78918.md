# #78918: Optimize wp-env source downloads with Git partial clones

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @elazzabi
- **Labels:** `[Type] Performance`, `[Tool] Env`
- **Merged:** [`b50c7db`](https://github.com/WordPress/gutenberg/commit/b50c7dbcd070b6753fc86611af5f5da43a6729d8)
- **Discussion:** [#78918](https://github.com/WordPress/gutenberg/pull/78918) · 4 comments · 0 reactions

## Summary

Adds `--filter=blob:none` to Git clone operations in `wp-env` to optimize source downloads. While `wp-env` previously used shallow clones (`--depth=1`) to skip Git history, file contents (blobs) were still downloaded upfront. This change introduces partial clones, allowing Git to defer fetching file contents until they are explicitly needed, significantly reducing fresh environment setup times.

## Impact

- **Local Development Tool Users**: Experience faster `wp-env start` and environment initialization, particularly noticeable on larger repositories like `WordPress/wordpress-develop` or slower networks.
- **Plugin & Theme Developers**: No action required. The underlying Git behavior remains fully compatible with existing workflows, sparse checkouts, and custom source configurations.
- **CI/CD & Automated Testing**: Build scripts relying on `wp-env` will see reduced setup latency without requiring environment variable changes or migration steps.

## Technical details

The change modifies the `git.clone` arguments used when downloading Git-based WordPress/plugin/theme/mapping sources. It appends `'--filter': 'blob:none'` to the existing shallow clone flags.

**Before:**
```js
await git.clone( source.url, source.clonePath, {
	'--depth': '1',
	'--no-single-branch': null,
} );
```

**After:**
```js
await git.clone( source.url, source.clonePath, {
	'--depth': '1',
	'--filter': 'blob:none',
	'--no-single-branch': null,
} );
```

This applies to all Git sources defined in `.wp-env.json` and specifically targets the `WordPress/wordpress-develop` checkout used for PHPUnit tests. The change relies on Git's lazy-fetch mechanism: commit/tree metadata is downloaded immediately, while blobs are fetched on-demand by subsequent Git operations (e.g., `git checkout <ref>`, `git show HEAD:path/to/file`, `git grep`). No external fetch scripts or manual post-clone steps are introduced.

## Contribution

Proposed and authored by @elazzabi, with co-authorship from @t-hamano. Merged in commit `b50c7db`. Labeled `[Type] Performance` and `[Tool] Env`, the discussion centered on framing the PR as either an enhancement or a performance fix, emphasizing the conservative approach to preserve existing tag/ref handling and sparse checkout logic. Benchmarks demonstrated a ~32% reduction in setup time (72s to 49s) for fresh environments without altering core clone/checkout behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
