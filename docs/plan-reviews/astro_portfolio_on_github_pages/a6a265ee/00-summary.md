# Plan Review Summary: Astro Portfolio on GitHub Pages

**Reviewer:** Cursor Agent (a6a265ee)
**Review date:** 2026-05-19
**Plan location:** `/Users/jerryluo/.cursor/plans/astro_portfolio_on_github_pages_6c1d5279.plan.md`

## Overall Verdict

REQUEST CHANGES

## Key Findings

- **Critical: Tailwind CSS 4 eliminates `tailwind.config.mjs`.** The plan lists `tailwind.config.mjs` in the project structure, but Tailwind CSS 4 uses CSS-first configuration via `@theme` directives inside CSS files. There is no JavaScript config file in v4. The project structure, and any implementation guidance that assumes a JS config, is wrong.
- **Critical: Astro 5 moved Content Collections config to `src/content.config.ts`.** The plan places the config at `src/content/config.ts`, which is the legacy Astro 3/4 location. In Astro 5, it must be `src/content.config.ts` at the project root's `src/` level, and collections require a `loader` (e.g. `glob()`) instead of the old `type: 'content'` property.
- **Major: Missing required `astro.config.mjs` settings for GitHub Pages.** The plan mentions this file but never specifies that `site` and (for non-username repos) `base` must be configured. Without `site`, RSS feeds via `@astrojs/rss` will fail at build time. Without `base`, assets on project-site deployments will 404.
- **Major: Astro is now at v6.x, not 5.x.** The plan pins "Astro 5.x" but the framework is currently at 6.1.2. This may cause version mismatches during `npm create astro` scaffolding and could mean the plan's assumptions about APIs don't hold for the latest version.
- **Major: GitHub Actions deploy action is at v6, not referenced.** The plan mentions `actions/deploy-pages` but the canonical approach is `withastro/action@v6`, which handles build + upload in a single step. Using the wrong action name will cause workflow failures.
- **Minor: No `output: 'static'` specified.** While Astro defaults to static output, explicitly setting `output: 'static'` in `astro.config.mjs` is best practice for GitHub Pages and should be called out.
- **Minor: No accessibility considerations.** The plan includes animations and visual design but never mentions WCAG compliance, semantic HTML, `prefers-reduced-motion` support for scroll animations, alt text for project thumbnails, or keyboard navigation.
- **Minor: No performance budget or Lighthouse targets.** For a portfolio site, performance is a key concern. The plan should set expectations.
- **Nit: No `.gitignore` in project structure.** Standard Astro scaffolding includes a `.gitignore` for `node_modules/`, `dist/`, `.astro/`, etc.

## Statistics

| Metric | Count |
|--------|-------|
| Plan docs reviewed | 1 |
| Critical findings | 2 |
| Major findings | 3 |
| Minor findings | 3 |
| Nits | 3 |
| Verified claims | 6 |
| Open questions | 5 |

## Recommendation

The plan should **not** proceed to implementation as-is. The two critical findings (Tailwind CSS 4 configuration model and Astro 5 Content Collections file location/API) would cause immediate build failures if implemented as described. These are not minor version drift issues — they represent fundamental API changes in both tools that affect project structure, configuration files, and code patterns throughout the implementation.

I recommend updating the plan to: (1) remove `tailwind.config.mjs` and document CSS-first configuration with `@tailwindcss/vite` or `@tailwindcss/postcss`, (2) move content collection config to `src/content.config.ts` with the `glob()` loader API, (3) specify required `astro.config.mjs` settings (`site`, `base`, `output`), (4) update the Astro version target to 6.x or at minimum verify all APIs against current docs, and (5) reference `withastro/action@v6` for the deploy workflow. Once these changes are made, the overall plan is sound and ready for implementation.
