# WordPress Contributor Toolkit 1.2: One app for your first Core or Gutenberg contribution

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** JuanMa Garrido
- **Published:** 2026-09-25
- **Tags:** `General`, `contributor day`, `playground`, `WordCamp`
- **Link:** [https://make.wordpress.org/core/2026/09/25/wordpress-contributor-toolkit-1-2-one-app-for-your-first-core-or-gutenberg-contribution/](https://make.wordpress.org/core/2026/09/25/wordpress-contributor-toolkit-1-2-one-app-for-your-first-core-or-gutenberg-contribution/)
- **Usefulness:** 2/5

## Summary

WordPress Contributor Toolkit v1.2.0, a desktop app for first-time contributors, now supports a full Gutenberg contribution workflow alongside the existing Core (Trac) workflow. When you create a site, you choose either WordPress Core or Gutenberg as the contribution target. A Gutenberg site clones the Gutenberg repository, builds it, and runs it in Playground. You can link a GitHub issue, apply existing PRs, and open a new PR from inside the app.

## Impact

- **New and first-time contributors / Contributor Day organizers:** A single app now covers setup, testing PRs, and submitting for both Core and Gutenberg. No action is required to keep using the Core workflow.
- **Existing Toolkit users:** Sites created before v1.1 are read-only. You can still read them and save their work as a patch, but you must create a new site to keep contributing. The app flags this on the site's card.
- **Gutenberg contributors:** A Gutenberg site is bound to that target at creation and cannot be changed afterwards. Gutenberg PR review expects testing steps, and the app's PR form has a field for them.
- **Plugin/theme developers, hosting, headless/REST consumers:** No effect. This is contributor tooling and changes no WordPress APIs.

## Technical details

**Gutenberg site setup**
- Clones the Gutenberg repo and installs dependencies with a bundled npm 11. It then builds the packages in one continuous setup.
- *Start dev server* runs stock WordPress in Playground with the checkout mounted as an already-activated Gutenberg plugin. Posts, settings and uploads persist across restarts.
- *Start build watch* runs Gutenberg's own `npm run dev`. It rebuilds everything once, then recompiles on each save.

**Issue and PR workflow**
- A GitHub issue card replaces the Trac ticket card. Each issue gets its own branch, `issue/N` (Core tickets use `ticket/N`).
- Linked issues list their fixing PRs with state. **Apply…** previews the touched files, then checks the PR out as `pr/N` with the author's commits. **Revert this PR** undoes it.
- **Review & submit changes** shows the full diff. Submitting signs in through GitHub's device flow, forks `WordPress/gutenberg`, pushes a branch and opens the PR through the API. The authorization is held in memory only and is discarded when the app quits. The app adds the `Fixes #N` line automatically.

**Git-based architecture (since v1.1)**
- The app now bundles a real Git client (the one GitHub Desktop uses) instead of the JavaScript reimplementation used in 1.0. It runs without your machine's Git settings.
- Site creation is a clone, ticket linking is a branch, patch application is `git apply`, trying a PR is a fetch and checkout, and updating trunk is a fetch and merge.
- Sites keep full history, fetched lazily, so `git log`, `git blame` and `git bisect` work.
- If you start a merge or rebase yourself, the app refuses to write until you finish or abandon it, and names the commands for either.
- A failing patch reports the regions that failed and leaves the checkout unchanged.

**Deep links (since v1.1)**
- The app handles `wpct://ticket/N` links. It asks before linking the ticket to the open site, because linking switches branches. A ticket link that arrives while a Gutenberg site is open waits for a Core site.
- On Windows, open the app once before using a link.
- The WP Trac Triager Chrome extension uses these links to open Trac tickets in the Toolkit.

## Contribution

The post credits review and feedback to @greenshady and @welcher. It gives no design discussion, rejected alternatives or debate. It does describe the switch from the JavaScript Git reimplementation in 1.0 to bundled native Git as the change behind the 1.1 and 1.2 workflow improvements, and it asks for feedback via GitHub issues, a feedback form, `#core` and `#core-editor`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
