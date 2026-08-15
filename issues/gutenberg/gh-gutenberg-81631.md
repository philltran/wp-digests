# #81631: Env: Fall back to cached docker images when the registry is unreachable

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Bug`, `[Tool] Env`
- **Merged:** [`0a886b4`](https://github.com/WordPress/gutenberg/commit/0a886b47c9640794db67ad8a60401eda1321c501)
- **Discussion:** [#81631](https://github.com/WordPress/gutenberg/pull/81631) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

wp-env now falls back to locally cached Docker images when the Docker registry cannot be reached during `wp-env start`. The previously unguarded image-pull step no longer aborts startup when the registry is unavailable. This fixes a hard failure that could occur when configuration changes or `--update` is used while the registry is down but cached images already exist.

## Impact

- **Plugin and theme developers using wp-env:** `wp-env start` no longer hard-fails at the image-pull step when the Docker registry is unreachable; it prints a notice and continues with cached images.
- **No action required:** No configuration, migration, or code changes are needed.
- **No breaking changes:** The change only makes the existing pull step non-fatal.

## Technical details

In `packages/env/lib/runtime/docker/index.js`, the `DockerRuntime` start path previously called `dockerCompose.pullAll( dockerComposeConfig );` directly. The diff wraps that call in a `try`/`catch` block. On failure, it calls `spinner.info( 'Could not pull docker images; using cached images instead.' )` and then continues to the next step, `spinner.text = 'Downloading sources.';`.

Before:

```js
await dockerCompose.pullAll( dockerComposeConfig );
```

After:

```js
try {
	await dockerCompose.pullAll( dockerComposeConfig );
} catch {
	spinner.info(
		'Could not pull docker images; using cached images instead.'
	);
}
```

The PR also adds a `packages/env/CHANGELOG.md` entry describing the fallback behavior.

## Contribution

The provided record contains no substantive design debate; the PR body notes that the fix and description were drafted with AI assistance and reviewed by the author.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
