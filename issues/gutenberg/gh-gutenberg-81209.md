# #81209: theme.json schema: fix block pseudo-classes and custom states

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `Global Styles`, `Backported to WP Core`
- **Merged:** [`ebab53f`](https://github.com/WordPress/gutenberg/commit/ebab53f3d33b2d6a0a23252f3111a3108c5e9211)
- **Discussion:** [#81209](https://github.com/WordPress/gutenberg/pull/81209) · 6 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

The `schemas/json/theme.json` JSON Schema now accepts the block-level states WordPress already supports: `:hover`, `:focus`, `:focus-visible` and `:active` on `core/button` and `core/navigation-link`, the `-current` custom state on `core/navigation-link`, and `elements` on `core/button`. Before this change, editors validating against the schema flagged these valid keys as errors. The cause was that the per-block rules were listed under `properties` but were also caught by a generic `patternProperties` fallback that rejected the extra keys.

## Impact

- **Theme developers / editors using the schema:** Valid `theme.json` files that use `:hover`/`:focus`/`:focus-visible`/`:active` on `core/button` or `core/navigation-link`, `-current` on `core/navigation-link`, or `elements` on `core/button` no longer produce schema warnings. Blocks without pseudo-class support (e.g. `core/paragraph`) still flag `:hover`.
- **Third-party blocks:** Namespaced blocks not listed under `properties` are validated by the fallback, which now applies through `additionalProperties`. The fallback is still `stylesPropertiesAndElementsComplete`, so pseudo-classes remain rejected on them.
- **Runtime behavior:** None. This changes only the JSON Schema, not how `WP_Theme_JSON` processes or sanitizes styles.
- **Action required:** None. Editors pointing `$schema` at `https://schemas.wp.org/trunk/theme.json` get the fix once the schema is published. It was also cherry-picked to `wp/7.1`.

## Technical details

**Root cause.** In `styles.blocks` and `styles.variations.*.blocks`, every core block was listed under `properties` and was also matched by a `patternProperties` entry (`^[a-z][a-z0-9-]*/[a-z][a-z0-9-]*$`). JSON Schema applies both, and the fallback's `propertyNames` rejected the extra keys. An `allOf` member can only narrow a key set, never widen it, so the more permissive per-block definitions never took effect.

**Changes in `schemas/json/theme.json`:**

- The fallback moves from `patternProperties` to `additionalProperties`, so it only applies to blocks not listed under `properties`. A `propertyNames` `pattern` still rejects malformed block names.

```json
// before
"patternProperties": { "^[a-z][a-z0-9-]*/[a-z][a-z0-9-]*$": { "$ref": "...stylesPropertiesAndElementsComplete" } },
"additionalProperties": false

// after
"propertyNames": { "pattern": "^[a-z][a-z0-9-]*/[a-z][a-z0-9-]*$" },
"additionalProperties": { "$ref": "...stylesPropertiesAndElementsComplete" }
```

  The same change is applied to the variation-level blocks map (`stylesVariationBlockPropertiesComplete`).
- New definitions:
  - `stylesBlocksPropertiesAndPseudoComplete`: style properties plus pseudo-selectors.
  - `stylesBlocksResponsiveSelectorsWithPseudoProperties`: `@mobile`/`@tablet` entries that allow the block's pseudo-selectors.
  - `stylesBlockVariationsWithPseudoProperties`: variations that allow properties, responsive states, pseudo-selectors, `elements` and `blocks`.
  - `stylesBlocksCustomStatesPropertyNames`: `enum: ["-current"]`.
- The `-current` state now references `stylesBlocksPropertiesAndPseudoComplete` instead of `stylesPropertiesComplete`, so a nested `:hover` inside `-current` validates.
- `core/button` gains `elements`, with a matching `propertyNames` allowlist. It uses the new responsive and variations definitions at block and variation level.
- `core/navigation-link` gets its own explicit key set (properties, responsive, pseudo, `-current`, `elements`, `variations`) at both block level and variation-block level, replacing the prior `stylesPropertiesAndElementsComplete` and `stylesVariationBlockPropertiesComplete` refs.

**Tests.** The new fixture `test/integration/fixtures/theme-json/block-states/theme.json` is rejected on trunk and accepted with this change. Several new expected-invalid fixtures under `test/integration/fixtures/schemas/` assert that these states are still refused where unsupported: `:hover` on `core/paragraph`, `:visited` under `@mobile` on `core/button`, `-other` on `core/navigation-link`, and pseudo-classes on a third-party block. Two existing fixtures that used `core/button` as the invalid example now use `core/paragraph`, since `core/button` gained its own definition. Run with `npm run test:unit -- test/integration/theme-schema.test.js`.

**Known gap.** `blocks["some/block"]["@mobile"].elements` is still rejected by the schema, although `WP_Theme_JSON_Gutenberg` sanitizes it and emits block nodes for it. This is tracked separately in #81253.

## Contribution

The PR closes #81057 and supersedes #81073. While working on it, @ramonjd found that `@mobile`-level `elements` is also rejected by the schema, and split that into follow-up #81253 because it concerns the responsive-states feature and touches all blocks. @t-hamano had no time to review it yet but asked for a backport to `wp/7.1`, since developers may refer to the schema on that branch. The bot then cherry-picked it there.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
