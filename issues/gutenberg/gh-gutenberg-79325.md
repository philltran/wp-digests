# #79325: api-fetch: Resolve to null for a 200 response with an empty body

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @MicahelE
- **Labels:** `[Type] Enhancement`, `[Package] API fetch`, `First-time Contributor`
- **Merged:** [`27cabdb`](https://github.com/WordPress/gutenberg/commit/27cabdb753346d8bee2da118c1c93a70a5fc7c39)
- **Discussion:** [#79325](https://github.com/WordPress/gutenberg/pull/79325) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`apiFetch` now resolves to `null` when a successful response has an empty body, instead of rejecting with `{ code: 'invalid_json' }`. Previously only `204` was treated as no-content; a `200` with an empty body failed in `response.json()`. Empty-body error responses (e.g. a `500` with no body) still reject with `invalid_json`.

## Impact

**Plugin & theme developers**
- Code that called `apiFetch` against endpoints returning `200` with no body and worked around the `invalid_json` rejection (e.g. a `catch` that checked `error.code === 'invalid_json'` to detect success) will now receive `null` from the resolved promise instead. Review such workarounds.
- Code that assumes a resolved value is always an object or array should handle `null` for these endpoints, as already needed for `204`.
- A non-empty body that isn't valid JSON still rejects with `invalid_json`.

**Site owners / hosting:** No action required.

**REST consumers:** Error responses are unchanged: an empty-body `4xx`/`5xx` still rejects with the `invalid_json` object, not `null`.

## Technical details

The change is in `packages/api-fetch/src/utils/response.ts`. `parseJsonAndNormalizeError` gains a second parameter, `allowEmptyBody = false`, and `parseResponseAndNormalizeError` (the success path) calls it with `true`.

The merged implementation differs from the approach shown in the PR description, which used `response.clone()` and only read the text after `json()` failed. Instead, the diff reads the body once with `response.text()`, returns `null` if `allowEmptyBody` is set and the text is `''`, and otherwise calls `JSON.parse( text )`. Any failure in the `try` block is normalized to `{ code: 'invalid_json', message: __( 'The response is not a valid JSON response.' ) }`. Response-likes without a `text()` function fall back to `response.json()`, preserving the prior behavior for mocks.

The `allowEmptyBody` flag exists because the error path does `throw await parseJsonAndNormalizeError( response )`. Returning `null` from the shared helper would have made empty error responses throw `null` rather than the `invalid_json` object that callers read `code` from.

```ts
// success path
return await parseJsonAndNormalizeError( response, true );
```

New tests in `src/test/index.jsdom.test.js` cover: `200` with empty body resolves to `null`; `200` with `'not json'` rejects with `invalid_json`; `500` with empty body rejects with `invalid_json`. A `CHANGELOG.md` Bug Fixes entry was added.

## Contribution

This is a first-time contributor PR that supersedes #62056, whose approach targeted the pre-TypeScript `response.js` and whose branch was reported to have no diff. Review stalled for a while before @mmtr, noting they were no longer a code owner, spotted that the shared helper would make empty error responses throw `null`; CodeRabbit flagged the same risk and the extra memory of cloning large successful responses. @Mamaduka asked for a rebase, and @MicahelE resolved the changelog conflict. The final version reworked the original clone-based design into a single `text()` read gated by `allowEmptyBody`, with a test for the empty error-body case.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
