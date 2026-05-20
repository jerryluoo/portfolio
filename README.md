# Portfolio

A modern portfolio website built with [Astro](https://astro.build) and [Tailwind CSS 4](https://tailwindcss.com), deployed to GitHub Pages.

**Live:** https://jerryluoo.github.io/portfolio/

## Features

- Zero JavaScript shipped by default (Astro islands architecture)
- Dark/light mode with system preference detection
- Responsive mobile-first design
- Scroll-triggered animations with `prefers-reduced-motion` support
- Blog powered by Astro Content Collections (Markdown)
- RSS feed
- SEO meta tags and Open Graph support
- Automatic deployment via GitHub Actions
- Lighthouse 90+ target across all categories

## Getting Started

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

## Customization

### Personal Information

Edit the following files to add your own content:

- `src/components/Hero.astro` — name, title, bio
- `src/components/Experience.astro` — work history
- `src/components/Projects.astro` — project showcase
- `src/components/Skills.astro` — technical skills
- `src/components/Education.astro` — education history
- `src/components/Contact.astro` — email and social links
- `src/components/Footer.astro` — footer social links

### Blog

Add Markdown files to `src/content/blog/` with this frontmatter:

```markdown
---
title: "Post Title"
pubDate: 2026-01-01
description: "A short description."
tags: ["tag1", "tag2"]
---

Post content here...
```

### Styling

Customize colors and fonts in `src/styles/global.css`:

```css
@theme {
  --color-primary: #3b82f6;
  --color-secondary: #6366f1;
  --font-sans: 'Inter', sans-serif;
}
```

### Deployment

1. Push to `main` branch
2. Enable GitHub Pages in repo **Settings > Pages > Source: GitHub Actions**
3. The site deploys automatically on every push

To change the deploy URL, update `site` and `base` in `astro.config.mjs`.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Astro 6.x |
| Styling | Tailwind CSS 4 |
| Blog | Content Collections + Markdown |
| Hosting | GitHub Pages |
| CI/CD | GitHub Actions |

## License

MIT
