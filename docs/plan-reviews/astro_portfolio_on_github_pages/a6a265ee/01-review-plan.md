# Review: Astro Portfolio on GitHub Pages

**Reviewer:** Cursor Agent (a6a265ee)
**Plan doc:** `/Users/jerryluo/.cursor/plans/astro_portfolio_on_github_pages_6c1d5279.plan.md`
**Review date:** 2026-05-19

## Verdict

REQUEST CHANGES

## Summary

The plan presents a sensible architecture for a greenfield Astro portfolio site deployed to GitHub Pages. The section breakdown, feature set, and hosting choice are well-considered. However, the plan contains two critical technical inaccuracies — Tailwind CSS 4's configuration model and Astro 5's Content Collections API — that would cause build failures if implemented as described. Several important deployment configuration details are also missing.

## Detailed Findings

### Finding 1: Tailwind CSS 4 eliminates JavaScript config files

**Severity:** Critical
**Location:** Project Structure section — `tailwind.config.mjs` listed as a top-level file; Stack section — "Tailwind CSS 4"
**Claim:** The plan lists `tailwind.config.mjs` as a project file and specifies Tailwind CSS 4.
**Reality:** Tailwind CSS 4 fundamentally changed its configuration model. There is no `tailwind.config.mjs` (or `.js`, `.ts`). Configuration is now CSS-first using `@theme` directives inside CSS files. The old `@tailwind base; @tailwind components; @tailwind utilities;` directives are replaced with a single `@import "tailwindcss";`. Additionally, Tailwind v4 uses `@tailwindcss/vite` (for Vite-based projects like Astro) or `@tailwindcss/postcss` instead of the old `tailwindcss` PostCSS plugin. Content detection is now automatic — no `content: [...]` array is needed.
**Recommendation:** Remove `tailwind.config.mjs` from the project structure. Add a `src/styles/global.css` (or similar) that uses `@import "tailwindcss";` and `@theme { ... }` for customization. Specify `@tailwindcss/vite` as the integration method in `astro.config.mjs`. Update the Stack section to note the CSS-first configuration approach.

### Finding 2: Astro 5 Content Collections config file location and API changed

**Severity:** Critical
**Location:** Project Structure section — `src/content/config.ts`; Blog section
**Claim:** The plan places the content collection config at `src/content/config.ts` and describes Content Collections for blog posts.
**Reality:** Astro 5 moved the content collection config file from `src/content/config.ts` to `src/content.config.ts` (at the `src/` level, not inside the `content/` directory). More significantly, the API changed: the `type: 'content'` property was replaced by a `loader` property. Filesystem content now requires importing `glob` from `astro/loaders` and configuring it with `pattern` and `base` options. The legacy `src/content/config.ts` location is deprecated and will be fully removed in Astro 6.
**Recommendation:** Update the project structure to show `src/content.config.ts`. Add a code snippet or description showing the new API pattern:
```typescript
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

const blog = defineCollection({
  loader: glob({ pattern: '**/*.md', base: './src/content/blog' }),
  schema: z.object({
    title: z.string(),
    pubDate: z.date(),
    description: z.string(),
    tags: z.array(z.string()).optional(),
  }),
});

export const collections = { blog };
```

### Finding 3: Missing required `astro.config.mjs` settings

**Severity:** Major
**Location:** Key Implementation Details — Section 1 (GitHub Pages Deployment); Project Structure
**Claim:** The plan lists `astro.config.mjs` as a project file and describes GitHub Pages deployment, but does not specify any required configuration.
**Reality:** GitHub Pages deployment requires setting `site` in `astro.config.mjs` (e.g. `site: 'https://<username>.github.io'`). For project-site deployments (repos not named `<username>.github.io`), the `base` option is also required (set to the repository name). Without `site`, `@astrojs/rss` will fail at build time. Without `base`, all asset paths on project sites will be wrong, causing 404 errors for CSS, JS, and images. Additionally, while Astro defaults to static output, explicitly setting `output: 'static'` is a best practice for deploy clarity.
**Recommendation:** Add a dedicated subsection specifying the required `astro.config.mjs` configuration:
```javascript
import { defineConfig } from 'astro/config';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  site: 'https://<username>.github.io',
  base: '/<repo-name>',  // omit for username.github.io repos
  output: 'static',
  vite: {
    plugins: [tailwindcss()],
  },
});
```

### Finding 4: Astro version is outdated

**Severity:** Major
**Location:** Stack section — "Astro 5.x"
**Claim:** The plan targets "Astro 5.x".
**Reality:** Astro is currently at version 6.1.2 (as of May 2026). Running `npm create astro@latest` will scaffold a v6 project. While most Astro 5 APIs carry forward, the plan should either target the current version or explicitly justify pinning to v5. Notably, Astro 6 removes legacy Content Collections support entirely, making Finding 2 even more critical.
**Recommendation:** Update the stack to "Astro 6.x" or "Astro (latest)" and verify all API references against current documentation. If there is a reason to pin to v5, document it.

### Finding 5: GitHub Actions workflow uses wrong action reference

**Severity:** Major
**Location:** Key Implementation Details — Section 1
**Claim:** The plan says the workflow will use "the official `actions/deploy-pages` action."
**Reality:** The canonical approach for deploying Astro to GitHub Pages uses `withastro/action@v6` (currently at v6.1.1), which handles both building the Astro project and uploading the artifact in a single step. `actions/deploy-pages` is the generic GitHub Pages deploy action that only handles the deployment step, not the build. While you could compose a workflow using `actions/upload-pages-artifact` + `actions/deploy-pages`, the `withastro/action` is simpler and officially recommended.
**Recommendation:** Reference `withastro/action@v6` as the deploy action. Provide a workflow snippet:
```yaml
name: Deploy to GitHub Pages
on:
  push:
    branches: [main]
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: withastro/action@v6
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

### Finding 6: No accessibility considerations

**Severity:** Minor
**Location:** Entire plan — absent
**Claim:** N/A — the plan does not mention accessibility.
**Reality:** A portfolio site is a public-facing professional artifact. Accessibility is both an ethical requirement and a practical one (screen readers, keyboard navigation, color contrast). The plan mentions "scroll-triggered fade-in animations" via Intersection Observer but does not mention `prefers-reduced-motion` support. It describes project card thumbnails but does not mention alt text requirements. Navigation is mentioned but keyboard focus management is not.
**Recommendation:** Add an accessibility section covering: semantic HTML structure (`<main>`, `<nav>`, `<section>`, `<article>`), `prefers-reduced-motion` media query for animations, alt text requirements for images, color contrast ratios for dark/light modes, keyboard navigability, and ARIA labels where needed.

### Finding 7: No performance targets

**Severity:** Minor
**Location:** Entire plan — absent
**Claim:** N/A — no performance goals mentioned.
**Reality:** Astro's strength is shipping zero JS by default, making excellent Lighthouse scores achievable. For a portfolio site, performance directly impacts user experience and SEO. The plan should set Lighthouse targets (Performance, Accessibility, Best Practices, SEO) and mention image optimization (Astro's built-in `<Image>` component or `astro:assets`).
**Recommendation:** Add a performance section with Lighthouse targets (e.g. 90+ across all categories) and mention using Astro's `<Image>` component for optimized images in the Projects section.

### Finding 8: No error handling for blog content

**Severity:** Minor
**Location:** Sections Overview — Blog; Key Implementation Details
**Claim:** Blog uses "Markdown-powered with frontmatter (title, date, description, tags)."
**Reality:** The plan does not describe what happens with malformed frontmatter, missing required fields, or empty blog collections. Astro's Content Collections with Zod schemas will throw build errors for invalid frontmatter, which is good, but the plan should document the expected schema and provide a sample blog post to make the expected format clear to future content authors.
**Recommendation:** Include the Zod schema for blog frontmatter and a sample `first-post.md` with complete frontmatter in the plan.

### Finding 9: Missing `.gitignore` in project structure

**Severity:** Nit
**Location:** Project Structure
**Claim:** The structure tree shows project files but omits `.gitignore`.
**Reality:** Every Astro project needs a `.gitignore` to exclude `node_modules/`, `dist/`, `.astro/` (the build cache directory), and `.env` files. Astro's scaffolding tool generates one automatically, but the plan's project structure should reflect it for completeness.
**Recommendation:** Add `.gitignore` to the project structure tree.

### Finding 10: Missing `tsconfig.json` in project structure

**Severity:** Nit
**Location:** Project Structure
**Claim:** The structure tree does not include `tsconfig.json`.
**Reality:** Astro projects include a `tsconfig.json` by default (even if not using TypeScript explicitly) because Astro uses TypeScript internally for content collection schemas and type-safe imports. The plan references `config.ts` for content collections, implying TypeScript usage.
**Recommendation:** Add `tsconfig.json` to the project structure tree.

### Finding 11: Contact form fallback not addressed

**Severity:** Nit
**Location:** Sections Overview — Contact
**Claim:** "optional contact form (using Formspree free tier for serverless form handling)"
**Reality:** The Formspree claim is accurate (50 submissions/month free), but the plan says the form is "optional" without specifying what the Contact section looks like without it. If the form is omitted, the section is just social links, which may be too thin to warrant its own section. The plan should clarify the default behavior.
**Recommendation:** Specify that the Contact section defaults to social links (GitHub, LinkedIn, email mailto) and the Formspree form is an opt-in enhancement that requires the user to create a Formspree account and add their form endpoint.

## Verified Claims

1. **GitHub Pages is free** — Confirmed. GitHub Pages provides free static hosting with 2,000 CI/CD minutes/month on free tier.
2. **View Transitions are built into Astro** — Confirmed. Astro has had View Transitions since v3, supporting fade, slide, and custom animations. The feature is active in current versions with known Safari 18 compatibility fixes applied (March 2026).
3. **Formspree free tier: 50 submissions/month** — Confirmed via Formspree documentation. The free tier includes unlimited forms, 30-day submission history, basic spam filtering, and reCAPTCHA.
4. **`@astrojs/rss` is the correct package for RSS feeds** — Confirmed. The package integrates with Content Collections via `getCollection()` and generates RSS XML at build time.
5. **Astro ships zero JS by default** — Confirmed. Astro's "islands architecture" only ships JavaScript for interactive components explicitly opted in via `client:*` directives.
6. **Custom domain configuration is free** — Confirmed. GitHub Pages supports custom domains via CNAME file and DNS configuration; the domain itself costs money but the Pages configuration is free.

## Questions for the Author

1. **Target Astro version:** Is the intent to use Astro 5.x specifically, or the latest stable version? This affects Content Collections API, Tailwind integration, and deploy action version choices.
2. **Project site vs. user site:** Will this be deployed to `<username>.github.io` (user site, no `base` needed) or a project site (requires `base` configuration)? This affects `astro.config.mjs` and internal link construction throughout all components.
3. **Dark mode implementation:** The plan mentions "dark mode toggle with system preference detection" but does not specify the implementation mechanism. Will this use Tailwind's `dark:` variant with a `class` strategy (requiring JS to toggle a class on `<html>`) or the `media` strategy (CSS-only, no toggle)? This is an architectural decision that affects every component.
4. **Blog pagination:** The plan describes a blog listing page but does not mention pagination. For a portfolio with potentially many posts, will pagination be needed? Astro supports `paginate()` in `getStaticPaths()` for this.
5. **Image assets:** Where will project thumbnails, profile photos, and other images live? The plan shows `public/favicon.svg` but does not describe an image asset strategy. Astro's `src/assets/` directory (with `astro:assets` for optimization) vs. `public/` (unprocessed) is an important architectural choice.
