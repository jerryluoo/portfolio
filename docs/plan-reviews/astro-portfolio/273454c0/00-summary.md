# Plan Review Summary: Astro Portfolio Testing Plan

**Reviewer:** Cursor Agent (273454c0)
**Review date:** 2026-05-19
**Plan location:** `docs/plans/astro-portfolio/`

## Overall Verdict

REQUEST CHANGES

## Key Findings

- **Major: `test-no-javascript-shipped` asserts at most 1 `<script>` tag, but the built site ships 3.** The BaseLayout has an inline dark mode script, Header.astro has a script for theme toggle + mobile menu, and index.astro has an IntersectionObserver script. The test assertion is wrong and will always fail.
- **Major: `test-meta-tags-present` asserts `og:image` exists on the index page, but it is never rendered.** BaseLayout only renders `og:image` when the `ogImage` prop is passed, and `index.astro` does not pass it. The test will always fail.
- **Major: `test-view-transitions` assumes View Transitions are implemented, but they are not.** No `ViewTransitions` component is imported or used anywhere in the codebase. The test describes behavior that does not exist.
- **Major: `test-image-optimization` checks for WebP/AVIF images and `srcset`, but the site has zero `<img>` tags.** Projects use inline SVG icons, not image thumbnails. There is no `src/assets/` directory. The test is irrelevant to the current implementation.
- **Major: RSS feed channel `<link>` is missing the `/portfolio/` base path.** `test-rss-feed-valid` checks that links include `jerryluoo.github.io/portfolio` but the channel-level `<link>` in the built RSS is `https://jerryluoo.github.io/` (no `/portfolio/`). This is an `@astrojs/rss` behavior that uses `site` without `base`.
- **Minor: `test-smooth-scroll-navigation` asserts "active nav item highlighted" but no active state highlighting is implemented** in Header.astro.
- **Minor: Edge case matrix references "skip-to-content link" but none exists** in BaseLayout or any component.
- **Minor: Edge case matrix references missing `src/assets/` image, but `src/assets/` does not exist.** The test scenario is valid conceptually but cannot be run.
- **Nit: No `test` script in `package.json`** and no test framework is installed. The plan describes shell-based tests but no mechanism to run them.
- **Nit: Regression test table says CI should run build verification on every commit, but the deploy workflow only runs on push to `main`** and has no separate lint/test job.

## Statistics

| Metric | Count |
|--------|-------|
| Plan docs reviewed | 1 |
| Critical findings | 0 |
| Major findings | 5 |
| Minor findings | 3 |
| Nits | 2 |
| Verified claims | 12 |
| Open questions | 4 |

## Recommendation

The testing plan has the right structure and coverage categories — build verification, content collections, accessibility, responsive design, SEO, dark mode, navigation, deployment, and performance. However, five major findings describe tests that will fail against the current implementation, not because of bugs in the code, but because the tests assert behavior the code does not provide or assert wrong thresholds. These should be corrected before the testing plan is used, otherwise developers will waste time debugging "test failures" that are actually spec mismatches.

I recommend: (1) fix the `<script>` count assertion to reflect the actual 3 scripts, (2) either implement `og:image` on the index page or remove the assertion, (3) either implement View Transitions or remove the test, (4) rework the image optimization tests to match the SVG-based project cards or add `<Image>` components first, and (5) investigate the RSS channel link base path issue. The remaining findings are minor and can be addressed alongside implementation.
