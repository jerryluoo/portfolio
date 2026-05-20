# Review: Astro Portfolio Testing Plan

**Reviewer:** Cursor Agent (273454c0)
**Plan doc:** `docs/plans/astro-portfolio/04-testing.md`
**Review date:** 2026-05-19

## Verdict

REQUEST CHANGES

## Summary

The testing plan is well-organized with comprehensive coverage categories spanning build verification, content collections, accessibility, responsive design, SEO, dark mode, navigation, deployment, and performance. The shell-based approach is practical for a static site. However, five of the test assertions contradict the actual codebase — they will fail not because of bugs, but because the tests describe behavior the implementation doesn't provide (View Transitions, image optimization, og:image) or assert incorrect thresholds (script count). These must be corrected before the plan can be used as a reliable test suite.

## Detailed Findings

### Finding 1: `test-no-javascript-shipped` asserts wrong script count

**Severity:** Major
**Location:** Section 1, `test-no-javascript-shipped` (lines 52–59)
**Claim:** "At most 1 inline `<script>` tag (the dark mode toggle). No external JS bundle references."
**Reality:** The built `dist/index.html` contains **3 `<script>` tags**:
1. `src/layouts/BaseLayout.astro` line 45: inline dark mode boot script (`is:inline`)
2. `src/components/Header.astro` line 81: theme toggle + mobile menu script (Astro-bundled)
3. `src/pages/index.astro` line 25: IntersectionObserver scroll animation script (Astro-bundled)

Verified by running `grep -c '<script' dist/index.html` which returns `3`. The assertion of "at most 1" will always fail. The overview plan (Section 3c) also states "Zero JS shipped by default; only dark mode toggle requires a small inline script" — this is also inaccurate.
**Recommendation:** Update the assertion to "At most 3 inline `<script>` tags (dark mode boot, theme toggle + mobile menu, scroll animations). No external JS bundle files in `dist/_astro/*.js`." Alternatively, audit whether the Header and IntersectionObserver scripts can be consolidated, but the test should match reality.

### Finding 2: `test-meta-tags-present` asserts `og:image` but it's never rendered on index

**Severity:** Major
**Location:** Section 5, `test-meta-tags-present` (lines 209–222)
**Claim:** Test checks for `og:image` on `dist/index.html` with assertion "All listed meta tags present on the index page with non-empty content."
**Reality:** In `src/layouts/BaseLayout.astro` line 31, `og:image` is conditionally rendered: `{ogImage && <meta property="og:image" content={ogImage} />}`. The index page (`src/pages/index.astro` line 13) renders `<BaseLayout title="Jerry Luo | Software Engineer">` with no `ogImage` prop. Running `grep 'og:image' dist/index.html` returns no matches. The test will always fail.
**Recommendation:** Either: (a) add a default `ogImage` to the BaseLayout that is always rendered (e.g. a profile photo or site screenshot), or (b) remove the `og:image` assertion from `test-meta-tags-present` and add a separate test that verifies `og:image` is present on pages that pass the prop. Option (a) is the better UX — social sharing without an OG image looks poor.

### Finding 3: `test-view-transitions` assumes View Transitions are implemented

**Severity:** Major
**Location:** Section 7, `test-view-transitions` (lines 309–314)
**Claim:** "A visible transition animation occurs between pages (fade, slide, or morph). No full page reload flash."
**Reality:** The codebase does not import or use Astro's `ViewTransitions` component anywhere. Searching all `.astro` files for `ViewTransitions`, `transition:`, or `astro:transitions` yields zero results. The overview plan (Section 3) lists "View Transitions (page transitions built into Astro)" as a feature, but it was never implemented. The test describes non-existent functionality.
**Recommendation:** Either: (a) implement View Transitions by adding `import { ViewTransitions } from 'astro:transitions'` and `<ViewTransitions />` in BaseLayout's `<head>`, then keep the test, or (b) remove the test and add a TODO to implement View Transitions later. The overview plan should also be updated to reflect the actual status.

### Finding 4: `test-image-optimization` tests are irrelevant to current implementation

**Severity:** Major
**Location:** Section 9, `test-image-optimization` (lines 375–383)
**Claim:** "Images in `dist/_astro/` are in WebP or AVIF format. `<img>` tags include `srcset` for responsive sizes."
**Reality:** The built site has **zero `<img>` tags** in `dist/index.html` (verified: `grep -c '<img' dist/index.html` returns 0). The Projects component (`src/components/Projects.astro`) uses inline SVG icons, not image thumbnails. There is no `src/assets/` directory. There are no images to optimize.
**Recommendation:** This test should be deferred until actual images are added to the site (e.g. project thumbnails, profile photo). Add a note: "This test is only relevant after images are added via `src/assets/` and the Astro `<Image>` component. Currently the site uses SVG icons for project cards." If images are planned, add them first, then the test becomes valid.

### Finding 5: RSS feed channel `<link>` missing base path

**Severity:** Major
**Location:** Section 5, `test-rss-feed-valid` (lines 236–245)
**Claim:** "Links include the full base URL with `/portfolio/` prefix."
**Reality:** The built `dist/rss.xml` has a channel-level `<link>` of `https://jerryluoo.github.io/` — note the missing `/portfolio/`. Individual item `<link>` elements correctly include `/portfolio/blog/first-post/`. This is because `rss.xml.ts` line 10 passes `site: context.site!.toString()`, which is the `site` config value without the `base` appended. The `@astrojs/rss` package does not automatically append `base` to the channel link.

The test command `rg '<link>' dist/rss.xml | rg 'jerryluoo.github.io/portfolio'` will match item links but the channel `<link>` will not match, which could cause confusion depending on how the assertion is evaluated.
**Recommendation:** Fix the RSS generation in `src/pages/rss.xml.ts` to include the base path in the site URL: `site: new URL(import.meta.env.BASE_URL, context.site!).toString()`. Then update the test to explicitly check both the channel `<link>` and item `<link>` elements separately.

### Finding 6: `test-smooth-scroll-navigation` asserts active nav highlighting

**Severity:** Minor
**Location:** Section 7, `test-smooth-scroll-navigation` (lines 289–294)
**Claim:** "Active nav item highlighted."
**Reality:** `src/components/Header.astro` does not implement active state highlighting. All nav links have the same static classes (`text-gray-600 dark:text-gray-300`). There is no IntersectionObserver or scroll spy that adds an active class to the current section's nav link. The smooth scrolling works (via `scroll-behavior: smooth` in `global.css` line 10), but active highlighting does not.
**Recommendation:** Either implement active nav highlighting (add a scroll spy that toggles an `active` class on nav links based on which section is in view) or remove the "Active nav item highlighted" assertion.

### Finding 7: Edge case matrix references skip-to-content link

**Severity:** Minor
**Location:** Section 10, Edge Case Matrix (line 409)
**Claim:** "Screen reader traversal: All sections announced with headings; skip-to-content link works."
**Reality:** There is no skip-to-content link in `BaseLayout.astro` or any component. The `<body>` immediately contains the `<slot />` with no skip link before the `<header>`. This is a WCAG 2.1 Level A requirement (Success Criterion 2.4.1).
**Recommendation:** Add a skip-to-content link as the first child of `<body>` in BaseLayout: `<a href="#main" class="sr-only focus:not-sr-only ...">Skip to content</a>`, and add `id="main"` to the `<main>` element. Then the edge case test becomes valid.

### Finding 8: Edge case references `src/assets/` but directory doesn't exist

**Severity:** Minor
**Location:** Section 10, Edge Case Matrix (line 404)
**Claim:** "Image file missing from `src/assets/` — Build fails with clear error message."
**Reality:** `src/assets/` does not exist in the project. The overview plan lists it in the project structure, but it was never created (no images are used). The edge case is conceptually valid but untestable.
**Recommendation:** Note this test is deferred until `src/assets/` is created with actual image files. When images are added, this becomes a valid regression test.

### Finding 9: No test runner or npm test script

**Severity:** Nit
**Location:** Entire plan — Section 11 (Manual Test Script)
**Claim:** The plan provides a bash script and individual shell commands as the test methodology.
**Reality:** `package.json` has no `test` script, and no test framework (Vitest, Playwright, Jest) is installed. The shell-based approach is practical for a static site, but there's no mechanism to run these tests — they exist only as documentation. The Section 11 bash script is useful but lives only in the plan doc, not as an executable file.
**Recommendation:** Add the Section 11 script as `scripts/test.sh` (executable), add `"test": "bash scripts/test.sh"` to `package.json`, and consider adding Vitest for the content collection validation tests (test-blog-invalid-frontmatter, test-blog-empty-collection) which currently require manual file manipulation.

### Finding 10: Regression tests assume CI runs on every commit

**Severity:** Nit
**Location:** Section 12, Regression Tests table (lines 486–492)
**Claim:** "Run When: Every commit (CI)" for build success, base path, and alt text checks.
**Reality:** The GitHub Actions workflow (`.github/workflows/deploy.yml`) only triggers on `push` to `main`. There is no separate CI job that runs on pull requests or non-main branches. The build verification happens implicitly during deploy, but failures would only be caught after merging to main — too late.
**Recommendation:** Either add a separate CI workflow that runs `npm run build` on pull requests (recommended), or update the regression table to say "Every push to main" instead of "Every commit."

## Verified Claims

1. **`test-build-succeeds` structure is correct** — `npm run build` producing `dist/index.html` is the right assertion. Verified: `dist/index.html` exists in the current build output.
2. **`test-static-output-structure` paths are correct** — Verified that `dist/index.html`, `dist/blog/index.html`, `dist/blog/first-post/index.html`, and `dist/rss.xml` all exist in the built output.
3. **`test-base-path-applied` approach is valid** — The built HTML does use `/portfolio/` prefixed paths for assets and links. Verified in `dist/index.html`.
4. **`test-blog-collection-valid` methodology is sound** — Building with valid frontmatter succeeds. The content collection schema in `src/content.config.ts` uses Zod and will throw build errors for invalid data.
5. **`test-blog-invalid-frontmatter` assertion is correct** — The Zod schema requires `title: z.string()` and `pubDate: z.date()`. A post with `title: 123` (number) and missing `pubDate` would fail validation. Verified against `src/content.config.ts` lines 8–13.
6. **`test-blog-empty-collection` is correctly handled** — `src/pages/blog/index.astro` checks `posts.length === 0` and renders an empty state message. Verified in the source.
7. **`test-semantic-html-structure` assertions match reality** — `dist/index.html` contains `<main>`, `<nav>`, `<footer>`, and multiple `<section>` elements. Verified.
8. **`test-dark-mode-toggle-aria` is correct** — `src/components/Header.astro` line 38 has `aria-label="Toggle dark mode"` on the theme toggle button. Verified.
9. **`test-prefers-reduced-motion` assertion is satisfied** — `src/styles/global.css` lines 13–26 contain a `@media (prefers-reduced-motion: reduce)` rule that disables animations. This appears in the built CSS bundle.
10. **`test-viewport-meta-tag` is correct** — `src/layouts/BaseLayout.astro` line 18 includes `<meta name="viewport" content="width=device-width, initial-scale=1.0" />`.
11. **`test-dark-mode-class-strategy` is correctly described** — BaseLayout uses `class="dark"` on `<html>`, Header.astro toggles the class via JS and persists to `localStorage`. System preference is respected as fallback.
12. **`test-blog-navigation` paths are correct** — Blog listing links to `/portfolio/blog/{id}/` and post pages can link back to `/portfolio/blog/`. Verified in source.

## Questions for the Author

1. **Should View Transitions be implemented or deferred?** The overview plan lists them as a feature, the testing plan includes a test for them, but they don't exist in the code. This needs a clear decision to avoid the test existing for non-existent functionality.
2. **What is the plan for images?** Multiple tests (image optimization, alt text, srcset) assume `<img>` tags exist, but the current site uses only SVG icons. If project thumbnails and a profile photo are planned, when will they be added relative to these tests?
3. **Should the shell-based tests be formalized into an executable script?** The Section 11 script is comprehensive but lives only in the plan document. Making it an actual `scripts/test.sh` would make these tests runnable.
4. **Should CI run on pull requests?** The regression table assumes "every commit" CI, but the workflow only runs on main. A PR-triggered build check would catch issues before merge.
