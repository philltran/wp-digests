# #81253: theme.json schema: responsive states belong to blocks

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `Global Styles`, `Backported to WP Core`
- **Merged:** [`197c370`](https://github.com/WordPress/gutenberg/commit/197c37094fea0aa68134bbb1f3a8096a40566a6a)
- **Discussion:** [#81253](https://github.com/WordPress/gutenberg/pull/81253) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `schemas/json/theme.json` JSON schema now matches what `WP_Theme_JSON` actually renders for responsive states. `elements` are accepted inside a block's `@mobile`/`@tablet` breakpoint, for every block including `core/button` and `core/navigation-link`. Breakpoints set directly on an element, at top level or block level, are now rejected because they produce no CSS.

## Impact

- **Theme developers / editor tooling:** `styles.blocks['core/group']['@mobile'].elements.link` no longer gets flagged by schema validation in editors that read the `$schema`. Previously that valid shape was rejected, while `elements.link['@mobile']` and `blocks.<block>.elements.link['@mobile']` were accepted even though they render nothing.
- **Existing themes:** any theme.json that puts `@mobile`/`@tablet` directly under an element (`styles.elements.link['@mobile']` or `styles.blocks.<block>.elements.link['@mobile']`) will now show schema errors. Those rules never produced output, so move them to `styles.blocks.<block>['@mobile'].elements.<element>`.
- **Runtime:** this is schema-only. No PHP or rendering change is included; the renderer fix for the block-level shape was a separate PR (#81265).
- **Not changed:** `-current` custom states still do not accept `elements`, since they have no element support in the renderer.

## Technical details

All changes are in `schemas/json/theme.json`, plus test fixtures.

**Added definitions**
- `stylesBlocksResponsiveStatePropertiesComplete`: `stylesProperties` plus an `elements` property referencing `stylesElementsPropertiesComplete`. A `propertyNames` constraint allows only style property names and `elements`, so `blocks` inside a breakpoint stays invalid.
- `stylesBlocksResponsiveStateWithPseudoPropertiesComplete`: the same, plus `stylesBlocksPseudoSelectorsProperties` and its property names, for blocks with pseudo-selector support.

**Rewired references**
- `stylesBlocksResponsiveSelectorsProperties` now points `@mobile`/`@tablet` at the first new definition (previously `stylesPropertiesComplete`).
- `stylesBlocksResponsiveSelectorsWithPseudoProperties` (used by `core/button` and `core/navigation-link`, introduced in #81209) now points at the second one (previously `stylesBlocksPropertiesAndPseudoComplete`).

**Removed**
- `stylesElementsResponsiveSelectorsProperties` and `stylesElementsResponsiveSelectorsPropertyNames`, along with every reference to them in the thirteen element definitions under `stylesElementsPropertiesComplete`.

`stylesBlocksPropertiesAndPseudoComplete` is intentionally unchanged because it is also the body of `-current`, which does not support `elements`.

**Shape now valid / invalid**
```jsonc
// valid
"core/group": { "@mobile": { "elements": { "link": { "color": { "text": "red" } } } } }
// invalid
"elements": { "link": { "@mobile": { "color": { "text": "red" } } } }
```

**Tests:** new valid fixture `test/integration/fixtures/theme-json/responsive-elements/theme.json` (covers `core/group`, `core/button`, `core/navigation-link`, a third-party block, style variations and shared variations). Invalid fixtures cover an element-level breakpoint (top-level and block-level), `blocks` inside a breakpoint (generic and pseudo variants), `-current.elements`, and an unknown element name. Run with `npm run test:unit -- test/integration/theme-schema.test.js`.

## Contribution

Opened by @ramonjd as a follow-up to #81209, which added responsive states for `core/button` and `core/navigation-link`. Review-thread discussion led to a separate renderer PR (#81265) so the block-level `@mobile.elements` shape would actually produce CSS. @talldan, using an AI agent's simulation, flagged that the branch predated #81209 and would leave those two blocks rejecting `elements` inside breakpoints. @ramonjd rebased and added the separate WithPseudo definition to cover them. Related follow-ups: #81312 (sanitizer) and #81309 (responsive states on block style variations).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
