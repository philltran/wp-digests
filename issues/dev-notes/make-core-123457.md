# PHP support clarification, spring 2026 edition

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** John Blackbourn
- **Published:** 2026-05-22
- **Tags:** `Core`, `PHP`, `PHP 8.0`, `php-compatibility`
- **Link:** [https://make.wordpress.org/core/2026/05/22/php-support-clarification-2026/](https://make.wordpress.org/core/2026/05/22/php-support-clarification-2026/)

## Summary

WordPress has retroactively removed the “beta” label from its PHP compatibility documentation, officially declaring full support for PHP 8.3, 8.4, and 8.5 across recent release branches. This policy clarification eliminates legacy conditional qualifiers that were adopted in 2023 based on adoption thresholds, explicitly confirming that PHP 8.x runs with minimal to no compatibility exceptions. The change is intended to remove friction for hosting providers and developers hesitant to upgrade server environments.

## Impact

- **Plugin & theme developers**: No code or API changes required; however, deferred testing against PHP 8.3+ can now proceed under officially documented full-support claims.
- **Hosting & platform teams**: Removes the “beta”/“compatible with exceptions” qualifier from official compatibility matrices, aligning server recommendations and automated environment checks with current ecosystem stability.
- **Site administrators/owners**: Clarifies that upgrading to PHP 8.4 or 8.5 is fully supported without expected breakage.
- **Action required**: None for core functionality; update internal documentation references and verify any custom environment assertions against the revised `Requirements` page.

## Technical details

This entry documents a policy and documentation shift rather than a code change. The legacy “beta” qualification—previously applied to PHP 8.x support pending a 10% adoption metric across WordPress versions—has been retroactively stripped from all release branches. Updated official documentation explicitly maps WordPress releases to full PHP support: WP 7.0/6.9 → PHP 8.5; WP 6.8+ → PHP 8.4; WP 6.4+ → PHP 8.3. The minimum supported version remains PHP 7.4 (established in WP 7.0), with PHP 8.3 designated as the recommended baseline. No core functions, hooks, constants, or schema definitions were modified; this strictly updates the handbook’s `Requirements` page and the official `php-compatibility` support matrix.

## Contribution

Authored by John Blackbourn and published to Make WordPress Core on May 22, 2026. The clarification followed community observation that the legacy “beta” label (originally adopted in 2023) was inadvertently delaying ecosystem upgrades due to perceived risk. Pre-publishing feedback from @garyj, @westonruter, and @jorbin confirmed that reported PHP compatibility exceptions remained statistically insignificant since WP 7.0 shipped, prompting the retrospective removal of the label across all documented release branches to accelerate PHP 8.x adoption.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
