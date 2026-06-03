# #78584: Add copilot-instructions.md file

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mikachan
- **Labels:** `[Type] Project Management`
- **Merged:** [`220a59c`](https://github.com/WordPress/gutenberg/commit/220a59c569ee5c3eebe8e657936633567ffa674e)
- **Discussion:** [#78584](https://github.com/WordPress/gutenberg/pull/78584) · 8 comments · 0 reactions

## Summary

A `.github/copilot-instructions.md` file was added to the Gutenberg repository to calibrate GitHub Copilot's PR review behavior repo-wide. The file instructs Copilot to focus only on semantic issues (bugs, logic errors, security, accessibility, API contract violations), skip style and formatting observations already enforced by lint and PHPCS, and suppress certain recurring false-positive patterns observed across real Copilot reviews in the repo. Three Gutenberg-specific guardrails prevent Copilot from second-guessing intentional architectural choices around `@wordpress/data`, WordPress i18n functions, and cross-package layering.

## Impact

**Contributors using GitHub Copilot for PR reviews in the WordPress/gutenberg repo:**
- Copilot reviews should produce fewer style/formatting comments, shorter inline notes, and less hallucinated WordPress core API feedback.
- Top-level review body will only appear when a finding genuinely spans multiple files; otherwise all findings should surface as inline comments.

**Plugin and theme developers / site owners / hosting & platform:**
- No action required. This is a contributor tooling change with no effect on released code, APIs, hooks, or block APIs.

## Technical details

The diff creates `.github/copilot-instructions.md` at 21 lines, organized in three sections that GitHub Copilot merges into every PR review when the file exists on the base branch:

**PR review guidelines**
- Only flag bugs, incorrect logic, security problems, accessibility regressions, or API contract violations.
- Skip style, formatting, naming, and whitespace (defer to lint/PHPCS).
- One or two sentences per comment; no code summaries.
- Top-level review body must remain empty unless a finding can't be attached to a specific line; never restate the diff.

**What not to flag**
- Do not speculate about WordPress, PHP, or Node APIs without verifying against the diff or repo files (`composer.json`, `package.json`). Do not invent strings, error messages, or behavior in WordPress core.
- Do not flag theoretical edge cases (`Number.MAX_VALUE`, subnormals, inputs unreachable from current callers).

**Gutenberg-specific context**
- Do not suggest replacing `@wordpress/data` selectors/actions with local React state.
- Do not suggest replacing `__()` / `_x()` / `_n()` calls with template literals.
- Do not suggest moving code across the `block-editor` → `editor` → `edit-post` package boundary without accounting for the layering rule (`block-editor` is WordPress-agnostic; lower layers must not depend on higher ones).

GitHub resolves `copilot-instructions.md` from the **base branch** of each PR, so the instructions take effect only after merge to trunk.

## Contribution

@mikachan opened the PR after sampling 50 recent Copilot reviews in the repo and using Claude Code to draft the initial instructions (disclosed in the PR). Two rules from the original draft — discouraging test-coverage suggestions and discouraging refactor-of-inherited-duplication suggestions — were removed following pushback from @aduth, who argued both are exactly the kind of feedback worth preserving. The top-level review-body rule was revised twice: the original "NEVER produce a review body" was softened first to a span-based exception, then refined to its final form. @mikachan validated the instructions by opening companion PR #78643 against this branch as base and requesting a Copilot review of deliberately flawed code, confirming the style/edge-case suppression rules fired correctly while the top-level-summary rule was only partially respected.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
