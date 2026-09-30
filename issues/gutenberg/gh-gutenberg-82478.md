# #82478: wp-env: Fix Docker build errors with Debian Bullseye repositories

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Tool] Env`, `Backported to WP Core`
- **Merged:** [`c8a1fce`](https://github.com/WordPress/gutenberg/commit/c8a1fcecaafc71cabc7d002d7bff86ef52175c59)
- **Discussion:** [#82478](https://github.com/WordPress/gutenberg/pull/82478) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`wp-env` Docker builds for the bullseye-based WordPress images (`wordpress:php7.4` and `wordpress:php8.0`) were failing with 404s on `apt-get -qy install sudo`, because Debian 11 reached end-of-life and its packages left the regular mirrors. The generated Dockerfile now points the `bullseye` apt suite at `archive.debian.org` and drops the `bullseye-security` and `bullseye-updates` entries, the same approach already used for `stretch` and `buster`.

## Impact

**Plugin/theme developers and CI maintainers**
- Anyone running `wp-env` with `phpVersion` set to `7.4` or `8.0` (or CI matrices that include those versions) could hit intermittent build failures, because the CDN was withdrawing bullseye packages node by node. This fix removes that failure mode.
- Newer PHP versions are not affected because they use non-bullseye images.
- No configuration change is needed. The fix ships in `@wordpress/env` once released. Until then, `wp-env` must be updated to a version containing this change.

**Caveats**
- Existing environments keep their cached Docker build layers. To pick up the fix locally, run `npm run wp-env destroy`, then `docker builder prune -af`, then `npm run wp-env start`. Note that `docker builder prune -af` clears the build cache for every project on the machine.
- Because `bullseye-security` and `bullseye-updates` are removed, these images will no longer receive security or update packages from apt in the wp-env build. That is reasonable for a local dev/test tool, but worth knowing.

## Technical details

The change is in `packages/env/lib/runtime/docker/docker-config.js`, in the Dockerfile template that rewrites `/etc/apt/sources.list` for EOL Debian releases. Three `RUN sed` lines were added after the `buster` block:

```dockerfile
# bullseye (https://www.debian.org/News/2026/20260831)
RUN sed -i 's|deb.debian.org/debian bullseye|archive.debian.org/debian bullseye|g' /etc/apt/sources.list
RUN sed -i '/bullseye-security/d' /etc/apt/sources.list
RUN sed -i '/bullseye-updates/d' /etc/apt/sources.list
```

Unlike `buster`, the security suite is deleted rather than rewritten to `archive.debian.org/debian-security`, because that archive does not yet carry `bullseye-security`, and rewriting it makes `apt-get update` fail with a 404 on the `Release` file. The resulting `sources.list` contains only `deb http://archive.debian.org/debian bullseye main` (plus commented snapshot lines). The `packages/env/CHANGELOG.md` gets an Unreleased Bug Fixes entry. No hooks, config schema, or public API changes.

## Contribution

@t-hamano opened and merged the PR himself to unblock CI failures in other PRs, noting it likely needs a backport to the 7.1 branch (the `Backported to WP Core` label is applied). The failure was intermittent and hard to reproduce locally, so the PR describes a cache-pruning procedure for verification. The PR disclosed that Claude Code was used to diagnose the cause and draft the change. The linked Debian end-of-life announcement is used in the comment because no `debian-devel-announce` post on bullseye archival existed yet.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
