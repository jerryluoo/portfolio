# 04 -- Astro Portfolio: Testing Plan

**Status:** Draft
**Depends on:** 00-overview (astro_portfolio_on_github_pages.plan.md)
**Blocks:** None

---

## 1. Build and Static Output Tests

These tests verify that the Astro project compiles without errors and produces correct static output.

### test-build-succeeds

Confirms the project builds to static HTML without errors.

```bash
npm run build
# Exit code must be 0
# dist/ directory must exist and contain index.html
```

**Assertion:** `dist/index.html` exists, exit code is 0, no error output on stderr.

### test-static-output-structure

Confirms all expected pages are generated.

```bash
# After build, verify these files exist:
test -f dist/index.html
test -f dist/blog/index.html
test -f dist/blog/first-post/index.html
test -f dist/rss.xml
```

**Assertion:** All listed paths exist in `dist/`.

### test-base-path-applied

Confirms all asset references use the `/portfolio` base path prefix.

```bash
rg -c 'src="/portfolio/' dist/index.html
rg -c 'href="/portfolio/' dist/index.html
# No bare absolute paths without /portfolio prefix for local assets
rg '"/(assets|styles|scripts)/' dist/index.html
```

**Assertion:** All local asset hrefs/srcs start with `/portfolio/`. No bare `/assets/`, `/styles/`, or `/scripts/` paths found.

### test-no-javascript-shipped

Confirms zero JS is shipped for pages without interactive islands.

```bash
# index.html should not include <script> tags (except the inline dark mode toggle)
rg '<script' dist/index.html | wc -l
```

**Assertion:** At most 1 inline `<script>` tag (the dark mode toggle). No external JS bundle references.

---

## 2. Content Collections Tests

### test-blog-collection-valid

Confirms the blog content collection loads correctly with valid frontmatter.

```bash
npm run build 2>&1 | rg -i "error"
```

**Assertion:** Build succeeds. No Zod validation errors in output.

### test-blog-invalid-frontmatter

Confirms the build fails gracefully when a blog post has invalid frontmatter.

Create a temporary file `src/content/blog/bad-post.md`:
```markdown
---
title: 123
description: "Missing pubDate field entirely"
---
Content here.
```

```bash
npm run build 2>&1
```

**Assertion:** Build fails with a Zod validation error mentioning `pubDate` (required) and `title` (expected string, received number). Remove `bad-post.md` after test.

### test-blog-empty-collection

Confirms the site builds even when the blog collection has no posts.

```bash
# Temporarily move all .md files out of src/content/blog/
mkdir /tmp/blog-backup
mv src/content/blog/*.md /tmp/blog-backup/
npm run build
mv /tmp/blog-backup/*.md src/content/blog/
rm -rf /tmp/blog-backup
```

**Assertion:** Build succeeds. `dist/blog/index.html` exists and renders an empty list (no crash).

---

## 3. Accessibility Tests

### test-lighthouse-accessibility

Run Lighthouse in CI or locally against the built site.

```bash
npx serve dist -l 4321 &
SERVER_PID=$!
sleep 2
npx lighthouse http://localhost:4321/portfolio/ \
  --output=json --output-path=./lighthouse-report.json \
  --only-categories=accessibility --chrome-flags="--headless --no-sandbox"
kill $SERVER_PID
```

**Assertion:** Accessibility score >= 90.

### test-semantic-html-structure

Confirm semantic landmarks exist on the index page.

```bash
rg '<main' dist/index.html
rg '<nav' dist/index.html
rg '<footer' dist/index.html
rg '<section' dist/index.html
```

**Assertion:** At least one `<main>`, one `<nav>`, one `<footer>`, and multiple `<section>` elements present.

### test-images-have-alt-text

Confirm all `<img>` tags have non-empty `alt` attributes.

```bash
# Find img tags without alt or with empty alt=""
rg '<img[^>]*(?!alt=)[^>]*>' dist/**/*.html
rg 'alt=""' dist/**/*.html
```

**Assertion:** No `<img>` without `alt`. No `<img>` with empty `alt=""` (except decorative images which should use `role="presentation"`).

### test-prefers-reduced-motion

Confirm that CSS respects `prefers-reduced-motion`.

```bash
rg "prefers-reduced-motion" dist/**/*.css
# OR in the built HTML inline styles
rg "prefers-reduced-motion" dist/index.html
```

**Assertion:** At least one `@media (prefers-reduced-motion: reduce)` rule exists that disables or reduces animations.

### test-dark-mode-toggle-aria

Confirm the dark mode toggle button has proper ARIA attributes.

```bash
rg 'aria-label' dist/index.html | rg -i "dark\|theme\|mode"
```

**Assertion:** A button element exists with an `aria-label` describing the dark mode toggle function.

---

## 4. Responsive Design Tests

### test-viewport-meta-tag

Confirm the viewport meta tag is present for mobile rendering.

```bash
rg 'viewport' dist/index.html
```

**Assertion:** `<meta name="viewport" content="width=device-width, initial-scale=1">` (or equivalent) present in `<head>`.

### test-responsive-breakpoints (Manual)

Open `http://localhost:4321/portfolio/` in a browser and verify at these widths:

| Viewport Width | Expected Behavior |
|----------------|-------------------|
| 320px (mobile) | Single column layout, hamburger nav, readable text |
| 768px (tablet) | Two-column project grid, expanded nav |
| 1024px (desktop) | Three-column project grid, full nav bar |
| 1440px (wide) | Content centered with max-width, no horizontal scroll |

**Assertion:** No horizontal overflow at any breakpoint. All text readable. Nav accessible at all sizes.

---

## 5. SEO and Meta Tags Tests

### test-meta-tags-present

Confirm essential SEO meta tags exist on every page.

```bash
rg '<title>' dist/index.html
rg 'meta.*description' dist/index.html
rg 'og:title' dist/index.html
rg 'og:description' dist/index.html
rg 'og:image' dist/index.html
rg 'og:url' dist/index.html
```

**Assertion:** All listed meta tags present on the index page with non-empty content.

### test-blog-post-meta-tags

Confirm blog posts have unique meta tags derived from frontmatter.

```bash
rg '<title>' dist/blog/first-post/index.html
rg 'meta.*description' dist/blog/first-post/index.html
```

**Assertion:** Title matches "Hello World" (or includes it). Description matches the frontmatter `description` field.

### test-rss-feed-valid

Confirm the RSS feed is valid XML with correct entries.

```bash
xmllint --noout dist/rss.xml
rg '<item>' dist/rss.xml
rg '<link>' dist/rss.xml | rg 'jerryluoo.github.io/portfolio'
```

**Assertion:** `rss.xml` is valid XML. Contains at least one `<item>`. Links include the full base URL with `/portfolio/` prefix.

### test-canonical-urls

Confirm canonical URLs use the correct site + base combination.

```bash
rg 'canonical' dist/index.html
rg 'canonical' dist/blog/first-post/index.html
```

**Assertion:** Canonical URLs start with `https://jerryluoo.github.io/portfolio/`.

---

## 6. Dark Mode Tests

### test-dark-mode-class-strategy (Manual)

1. Load the page in a browser.
2. Click the dark mode toggle.
3. Inspect `<html>` element.

**Assertion:** `<html>` element gains `class="dark"` when toggled. Colors invert appropriately.

### test-dark-mode-persists (Manual)

1. Toggle dark mode on.
2. Navigate to a blog post.
3. Navigate back to home.

**Assertion:** Dark mode preference persists across page navigations (stored in `localStorage`).

### test-dark-mode-system-default (Manual)

1. Set OS to dark mode.
2. Load the page with cleared localStorage.

**Assertion:** Site renders in dark mode without user interaction (respects `prefers-color-scheme: dark`).

---

## 7. Navigation and Routing Tests

### test-smooth-scroll-navigation (Manual)

1. Click each nav link (Hero, Experience, Projects, Skills, Education, Contact).
2. Observe scroll behavior.

**Assertion:** Page scrolls smoothly to the target section. URL hash updates. Active nav item highlighted.

### test-blog-navigation

Confirm blog pages are navigable.

```bash
# Blog listing links to posts
rg 'href="/portfolio/blog/first-post' dist/blog/index.html
# Blog post links back to blog listing
rg 'href="/portfolio/blog"' dist/blog/first-post/index.html
```

**Assertion:** Blog listing links to individual posts. Posts link back to the listing.

### test-view-transitions (Manual)

1. Navigate from the home page to a blog post.
2. Navigate back.

**Assertion:** A visible transition animation occurs between pages (fade, slide, or morph). No full page reload flash.

---

## 8. Deployment Pipeline Tests

### test-github-actions-workflow-syntax

Validate the workflow YAML is syntactically correct.

```bash
npx yaml-lint .github/workflows/deploy.yml
# OR use actionlint if available
actionlint .github/workflows/deploy.yml
```

**Assertion:** No YAML syntax errors. No undefined action references.

### test-deploy-succeeds-on-push

After pushing to `main`:

1. Go to GitHub repo > Actions tab.
2. Verify the "Deploy to GitHub Pages" workflow triggered.
3. Verify both `build` and `deploy` jobs succeed (green checkmarks).
4. Visit `https://jerryluoo.github.io/portfolio/`.

**Assertion:** Workflow completes successfully. Site is accessible at the expected URL.

### test-deploy-assets-resolve

After deployment, verify assets load correctly on the live site.

```bash
curl -s -o /dev/null -w "%{http_code}" https://jerryluoo.github.io/portfolio/
curl -s https://jerryluoo.github.io/portfolio/ | rg '404' | wc -l
```

**Assertion:** HTTP 200. No 404 errors in browser console for CSS, JS, images, or fonts.

---

## 9. Performance Tests

### test-lighthouse-performance

```bash
npx serve dist -l 4321 &
SERVER_PID=$!
sleep 2
npx lighthouse http://localhost:4321/portfolio/ \
  --output=json --output-path=./lighthouse-perf.json \
  --only-categories=performance,best-practices,seo \
  --chrome-flags="--headless --no-sandbox"
kill $SERVER_PID
cat lighthouse-perf.json | npx json categories.performance.score
```

**Assertion:** Performance >= 90, Best Practices >= 90, SEO >= 90.

### test-image-optimization

Confirm images are served in modern formats.

```bash
rg 'srcset' dist/index.html
rg '\.webp\|\.avif' dist/_astro/ 2>/dev/null | wc -l
```

**Assertion:** Images in `dist/_astro/` are in WebP or AVIF format. `<img>` tags include `srcset` for responsive sizes.

### test-page-weight

Confirm total page weight is reasonable for a portfolio site.

```bash
du -sh dist/
find dist/ -name "*.html" -exec wc -c {} + | tail -1
```

**Assertion:** Total `dist/` size under 5MB (excluding large image assets). Individual HTML pages under 100KB.

---

## 10. Edge Case Matrix

| Scenario | Expected Behavior |
|----------|-------------------|
| Blog collection has 0 posts | Build succeeds; blog listing shows empty state message |
| Blog post has no `tags` field | Build succeeds; tags section hidden on that post |
| Image file missing from `src/assets/` | Build fails with clear error message pointing to missing file |
| `site` removed from `astro.config.mjs` | Build fails (RSS generation requires `site`) |
| `base` removed from `astro.config.mjs` | Build succeeds but assets 404 on GitHub Pages project site |
| Very long blog post (10,000+ words) | Build succeeds; page renders without layout overflow |
| Blog post title with special characters (`<`, `&`, quotes) | Build succeeds; title properly escaped in HTML and RSS |
| User disables JavaScript | Site fully readable (only dark mode toggle non-functional) |
| Screen reader traversal | All sections announced with headings; skip-to-content link works |
| 404 page request | GitHub Pages shows default 404 or custom 404.html if provided |
| Concurrent blog posts with same date | Build succeeds; both posts appear in listing sorted by title |

---

## 11. Manual Test Script

A developer can run this end-to-end after implementation:

```bash
#!/bin/bash
set -e

echo "=== 1. Install dependencies ==="
npm install

echo "=== 2. Run dev server (verify no startup errors) ==="
npm run dev &
DEV_PID=$!
sleep 5
curl -s -o /dev/null -w "%{http_code}" http://localhost:4321/portfolio/
kill $DEV_PID

echo "=== 3. Build for production ==="
npm run build

echo "=== 4. Verify output structure ==="
test -f dist/index.html && echo "PASS: index.html exists"
test -f dist/blog/index.html && echo "PASS: blog/index.html exists"
test -f dist/blog/first-post/index.html && echo "PASS: blog post exists"
test -f dist/rss.xml && echo "PASS: RSS feed exists"

echo "=== 5. Verify base path in assets ==="
if rg -q '"/portfolio/' dist/index.html; then
  echo "PASS: base path applied"
else
  echo "FAIL: base path missing"
  exit 1
fi

echo "=== 6. Verify semantic HTML ==="
rg -q '<main' dist/index.html && echo "PASS: <main> present"
rg -q '<nav' dist/index.html && echo "PASS: <nav> present"
rg -q '<footer' dist/index.html && echo "PASS: <footer> present"

echo "=== 7. Serve and run Lighthouse ==="
npx serve dist -l 4321 &
SERVE_PID=$!
sleep 2
npx lighthouse http://localhost:4321/portfolio/ \
  --output=json --output-path=./lighthouse.json \
  --chrome-flags="--headless --no-sandbox" \
  --quiet
kill $SERVE_PID

echo "=== 8. Check Lighthouse scores ==="
node -e "
const r = require('./lighthouse.json');
const cats = r.categories;
const pass = (name, score) => score >= 0.9 ? 'PASS' : 'FAIL';
console.log(pass('Performance', cats.performance.score) + ': Performance ' + (cats.performance.score * 100));
console.log(pass('Accessibility', cats.accessibility.score) + ': Accessibility ' + (cats.accessibility.score * 100));
console.log(pass('Best Practices', cats['best-practices'].score) + ': Best Practices ' + (cats['best-practices'].score * 100));
console.log(pass('SEO', cats.seo.score) + ': SEO ' + (cats.seo.score * 100));
"

echo "=== All tests complete ==="
```

---

## 12. Regression Tests

Since this is a greenfield project, there are no pre-existing tests. Once implemented, the following should be maintained as regression guards:

| Test | Purpose | Run When |
|------|---------|----------|
| `npm run build` succeeds | Catch any breaking config or content change | Every commit (CI) |
| Lighthouse scores >= 90 | Prevent performance regressions | Weekly or per-release |
| RSS feed valid XML | Catch blog schema changes that break feed | Every blog post addition |
| No bare absolute paths without base | Catch routing regressions | Every commit (CI) |
| All images have alt text | Catch accessibility regressions | Every commit (CI) |
