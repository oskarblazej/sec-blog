# oskarblazej.tech

A modern, performance-focused security blog built with Astro. Technical deep-dives into application security and offensive security research.

## Credits

This project is based on [klapacz/blog](https://github.com/klapacz/blog) — an Astro blog template by [Klapacz](https://klapacz.dev).

Modified and customized for security-focused content under the MIT License.

## 🛠 Tech Stack

- **Framework:** Astro 5 — Static site generation with zero JavaScript by default
- **Styling:** Tailwind CSS v4 with `@tailwindcss/typography` for rich prose
- **Content:** MDX with syntax highlighting via `shiki` and `rehype-pretty-code`
- **UI Components:** React (optional integration) + shadcn/ui patterns
- **Dark Mode:** System preference detection with localStorage persistence
- **SEO:** Automatic sitemap generation, RSS feed, Open Graph metadata

## ✨ Features

- **Fast:** Static HTML generation, minimal JavaScript, optimized bundle
- **Dark Mode:** Automatic theme detection with smooth transitions
- **Responsive Design:** Mobile-first, Tailwind-based layout
- **Blog System:** Content collection (`src/content/blog/`) with automatic routing
- **Code Highlighting:** Custom syntax highlighting with diff annotations
- **Type Safe:** Full TypeScript support, strict mode enabled
- **RSS Feed:** Auto-generated RSS at `/rss.xml`
- **Sitemap:** Auto-generated sitemap at `/sitemap-index.xml`

## 📁 Project Structure

```
src/
├── pages/              # Route definitions
│   ├── index.astro     # Home page
│   └── blog/           # Blog section
├── components/         # Reusable UI components
├── layouts/            # Page layouts (BlogPost, etc)
├── content/            # Markdown blog posts
│   └── blog/           # Blog collection
├── styles/             # Global CSS
└── consts.ts           # Site configuration

public/                 # Static assets (favicon, fonts)
astro.config.mjs        # Astro configuration
tailwind.config.js      # Tailwind CSS configuration
```

## 🚀 Getting Started

### Prerequisites
- Node.js 18+
- npm or pnpm

### Development

```bash
npm install
npm run dev        # Start dev server at http://localhost:3000
```

### Building

```bash
npm run build      # Build static site (output: ./dist/)
npm run preview    # Preview production build locally
```

## 📄 License

MIT — Use freely for personal and commercial projects.
