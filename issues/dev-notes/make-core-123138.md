# Dev Chat summary: May 6, 2026

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Jb Audras
- **Published:** 2026-05-13
- **Tags:** `Devchat`, `General`, `Summaries`, `7.0`, `core`, `core-editor`, `dev chat`
- **Link:** [https://make.wordpress.org/core/2026/05/13/summary-dev-chat-may-6-2026/](https://make.wordpress.org/core/2026/05/13/summary-dev-chat-may-6-2026/)

## Summary

WordPress 7.0 RC3 is scheduled for May 8, finalizing the Real-Time Cache (RTC) introduction process and advancing documentation efforts like the Field Guide. Concurrently, the RT custom table schema has cleared systems team review, locking database changes for release while explicitly rejecting further procedural formalization for similar future requests.

## Impact

- **Platform & Hosting Teams**: RC3 artifacts will be available May 8; prepare infrastructure for RTC testing and verify compatibility with the finalized custom table structure.
- **Core Contributors & Plugin/Theme Developers**: The Field Guide is under final review for immediate publication to aid new contributors. No code migrations, API changes, or configuration adjustments are required for developers at this stage.

## Technical details

The RC3 build incorporates the finalized approach from #64696 regarding RTC introduction, completing database schema additions validated by the systems team. Testing infrastructure and a standalone plugin are ready for production-scale validation without modifying trunk core files. Development effort is now concentrated on closing out remaining milestone 7.0 tickets (notably clarifying status for #64536) rather than introducing new hooks, block schemas, or REST endpoints.

## Contribution

Synthesized from the May 6, 2026 dev chat summary coordinated by @audrasjb, with active tracking by @desrosj and @joefusco. A proposal to document a formalized systems review process for future schema changes was raised but explicitly rejected by attendees as insufficiently frequent to warrant. Dedicated Gutenberg scrubbing sessions were organized by @juanmaguitar to accelerate milestone 7.0 closure.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
