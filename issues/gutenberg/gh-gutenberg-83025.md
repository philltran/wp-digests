# #83025: Stylelint: add `custom-property-pattern` to disallow `--_gcd-*` custom properties

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @simison
- **Labels:** `[Type] Enhancement`, `[Tool] stylelint config`, `[Package] UI`
- **Merged:** [`1628e62`](https://github.com/WordPress/gutenberg/commit/1628e624018571bab37bc214301446e72fe0bf33)
- **Discussion:** [#83025](https://github.com/WordPress/gutenberg/pull/83025) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/stylelint-config` package now ships a `custom-property-pattern` rule that flags any use or definition of `--_gcd-*` custom properties as an error. These tokens are a private internal bridge in `@wordpress/ui` for wp-admin CSS defense, not a public theming API, and the rule prevents external plugins and themes from accidentally depending on them. The `@wordpress/ui` package itself opts out via a local `.stylelintrc.mjs` that sets the rule to `null`.

## Impact

- **Plugin & theme developers using `@wordpress/stylelint-config`:** Any CSS that sets or references a `--_gcd-*` variable will now fail `npm run lint:css`. If you have been overriding these tokens to force a design, you must remove those overrides.
- **Projects that already define their own `custom-property-pattern`:** Stylelint does not merge this rule. Your existing pattern silently replaces the preset's, so the `--_gcd-*` ban is lost. You must manually add a `(?!_gcd-)` negative lookahead to your own pattern to retain the restriction.
- **`@wordpress/ui` maintainers:** No action required; the package ships a local `.stylelintrc.mjs` that disables the rule.
- **No runtime or API change.** This is purely a lint-time guard; no CSS output, JS API, or REST surface is affected.

## Technical details

In `packages/stylelint-config/index.js`, a new entry is added to the exported rules object:

```js
'custom-property-pattern': [
  '^(?!_gcd-).+',
  {
    message: ( name ) =>
      `Do not use "${ name }". \`--_gcd-*\` variables are an internal @wordpress/ui global CSS defense detail.`,
  },
],
```

The regex `^(?!_gcd-).+` is matched against the custom-property name *without* the leading `--`, so it rejects any name beginning with `_gcd-`.

A new file `packages/ui/.stylelintrc.mjs` is created to scope-disable the rule for the `@wordpress/ui` package:

```js
export default {
  extends: '@wordpress/stylelint-tools/config',
  rules: {
    'custom-property-pattern': null,
  },
};
```

Test fixtures (`custom-property-pattern-invalid.css`, `custom-property-pattern-valid.css`) and a Vitest suite (`custom-property-pattern.js`) with a snapshot verify that both a declaration (`--_gcd-heading-font-weight: 600`) and a `var()` reference produce an error, while a normal `--my-token` does not.

The README gains a dedicated `### custom-property-pattern` section documenting the non-merge behavior and showing the manual-merge pattern for projects that already set the rule.

## Contribution

Opened by @simison as a lighter-weight alternative to the earlier PR #80952 (a custom Stylelint plugin rule). During review, @mirka suggested also banning `--_wp-` prefixed tokens; @simison agreed the idea was sound but deferred it to a separate PR because `packages/widget-dashboard` and `packages/grid` still use those prefixes internally. The PR was merged with @mirka as co-author. AI tooling was disclosed in the PR body.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
