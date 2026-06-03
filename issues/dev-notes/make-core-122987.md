# Proposal: Auto-generate Block Editor Handbook docs from block.json

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** JuanMa Garrido
- **Published:** 2026-05-05
- **Tags:** `General`, `block editor handbook`, `block-editor`, `blocks`, `docs`, `handbooks`
- **Link:** [https://make.wordpress.org/core/2026/05/05/proposal-auto-generate-block-editor-handbook-docs-from-block-json/](https://make.wordpress.org/core/2026/05/05/proposal-auto-generate-block-editor-handbook-docs-from-block-json/)

## Summary

A proposed automated pipeline to generate per-block API reference pages for the Block Editor Handbook directly from each block's `block.json`. This change aims to replace scattered documentation lookups with a structured, always-current developer guide that links attributes, supports, and context in handbooks. Currently tracked as PR #77612 following an initial merged-but-reverted attempt (#77350/#77590), the proposal introduces build-time generation of `README.md` files alongside block source code to improve both developer experience and AI/LLM parsing capabilities.

## Impact

['**Block & Plugin Developers**: Per-block documentation will automatically surface attributes, supports, selectors, and example markup derived from `block.json` and test fixtures. Manual narrative additions can be preserved alongside auto-generated reference data using token delimiters.', '**Handbook Contributors & Docs Maintainers**: API reference tables no longer require manual maintenance or separate PRs when block metadata changes. Focus shifts toward narrative guides, tutorials, and accessibility notes.', '**AI/LLM Integration**: Structured markdown output per block improves semantic context for automated template, pattern, and block markup generation, reducing token consumption and hallucination risks.', '**Action Required**: No immediate action required; this remains an active proposal (PR #77612). Once merged, the docs build pipeline will need to be run locally or via CI to generate initial files.']

## Technical details

["Introduces a new documentation script (`docs-blocks`, referenced as 'Docs Block Detail') added to the Gutenberg `scripts` family in `package.json`, triggered alongside existing `lint-staged` and API doc workflows.", "Generates a `README.md` inside each block's source directory (e.g., `packages/block-library/src/{block}/README.md`).", 'Auto-generated content is wrapped in `<!-- START TOKEN / END TOKEN -->` delimiters, preserving any hand-written prose above or below the markers during regeneration.', 'Reference pages extract metadata from `block.json` and link to relevant Handbook sections (attributes, supports). Block classification (static, dynamic, hybrid) is inferred automatically by checking for specific file existence within the block directory.', 'Example markup is pulled directly from internal test fixtures used for block verification.', 'Output paths follow `developer.wordpress.org/block-editor/reference-guides/core-blocks/[block-category]/[block-name]`, mirroring the existing pattern used by `update-api-docs.js` for package READMEs.']

## Contribution

['Author **JuanMa Garrido** initially implemented a similar pipeline (PR #77350), which was merged and subsequently reverted as #77590 due to insufficient community discussion. This refined proposal, PR #77612, was published on May 5, 2026.', 'A **Hallway Hangout** was hosted on May 18, 2026, with community feedback collected through May 25. Key reviewers included **Birgit Pauli-Haack**, **Andrea Roenning**, and **Joe Dawson**.', 'Design consensus favored keeping generated `README.md` files inside the repository next to source code rather than generating them solely at the docs site level, ensuring availability via both GitHub and the handbook.', 'Deferred considerations include adding multi-category grouping for deprecated blocks, displaying specific Gutenberg/Core versioning info, and auto-generating changelogs. These were explicitly noted as valuable but scoped out of this initial PR to avoid bloating the diff; next steps are pending post-feedback review.']

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
