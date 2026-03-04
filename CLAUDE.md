# CLAUDE.md — AI Assistant Guide for maravexa.com

This file describes the codebase, conventions, and workflows for
[maravexa.com](https://maravexa.com), a personal website owned by Mara Vexa.

---

## Project Overview

A **zero-JavaScript static site** built with [Zola](https://www.getzola.org/)
(Rust-based SSG) and deployed to **Cloudflare Pages** via GitHub push. The
design philosophy prioritises performance, security, accessibility, and
maintainability above all else.

Key constraints to never violate:
- **No JavaScript** — not even a single `<script>` tag in any template
- **No external requests** — no CDN fonts, no analytics, no third-party assets
- **No backend** — pure static output, no server-side logic

---

## Tech Stack

| Layer | Tool | Notes |
|---|---|---|
| SSG | Zola 0.19.2 | Single binary, no npm/pip dependencies |
| Templates | Tera (Jinja2-like) | `.html` files in `templates/` |
| Content | Markdown + TOML frontmatter | Files in `content/` |
| Styles | Vanilla CSS | One file: `static/style.css` |
| Hosting | Cloudflare Pages | Auto-deploy on push to GitHub |
| Security | `static/_headers` | Cloudflare-native security header rules |

---

## Repository Structure

```
├── config.toml              # Zola site configuration (base_url, feeds, taxonomies)
├── content/
│   ├── _index.md            # Homepage content (Markdown)
│   ├── about.md             # /about page
│   └── blog/
│       ├── _index.md        # Blog section settings (sort, paginate)
│       └── *.md             # One file per blog post
├── static/
│   ├── style.css            # Entire stylesheet (~5 KB, ~400 lines)
│   ├── _headers             # Cloudflare Pages security/cache headers
│   ├── _redirects           # Cloudflare URL redirects
│   ├── robots.txt           # Crawler policy
│   ├── llms.txt             # AI crawler info / site summary
│   └── img/                 # Images — always provide .webp AND .jpg pairs
└── templates/
    ├── base.html            # Master HTML shell (head, header, footer)
    ├── index.html           # Homepage template
    ├── page.html            # Generic static page (About, etc.)
    ├── blog-list.html       # Post listing / archive
    ├── blog-page.html       # Single blog post (title, date, tags, prev/next)
    ├── taxonomy_list.html   # /tags/ overview
    ├── taxonomy_single.html # /tags/<tag>/ filtered list
    └── shortcodes/
        └── img.html         # {{ img(...) }} shortcode — WebP + JPEG fallback
```

---

## Development Commands

```bash
# Install Zola (one-time)
brew install zola          # macOS
pacman -S zola             # Arch Linux
snap install zola --edge   # Ubuntu

# Run local dev server (live-reloads on save)
zola serve
# → http://127.0.0.1:1111

# Production build (output in /public)
zola build
```

There is no `package.json`, `Makefile`, or test suite. The build is `zola build`
and nothing else.

---

## Content Conventions

### Blog Post Frontmatter

Every blog post in `content/blog/` must have this TOML frontmatter block:

```toml
+++
title = "Post Title"
date = 2026-03-01
description = "One-sentence summary for the post list and SEO meta tag."
[taxonomies]
tags = ["tag1", "tag2"]
+++

Post body in Markdown follows here.
```

- `date` — ISO 8601 (`YYYY-MM-DD`). Zola uses this for sorting.
- `description` — required; shown in post lists and `<meta name="description">`.
- `tags` — array of lowercase strings; drives the `/tags/` taxonomy pages.
- Do **not** add `draft = true` to published posts.

### Image Workflow

1. Place both `.webp` and `.jpg` versions in `static/img/`.
2. Target **< 150 KB** per image.
3. Convert with: `cwebp -q 82 photo.jpg -o photo.webp`
4. Reference in Markdown using the `img` shortcode — **never** raw `<img>` tags:

   ```
   {{ img(src="photo.webp", alt="Describe the image for screen readers", caption="Optional caption", w=800, h=533) }}
   ```

   The `w` and `h` attributes are **required** to prevent Cumulative Layout Shift (CLS).

---

## Template Conventions

### Tera Template Engine

- Inherits via `{% extends "base.html" %}` and `{% block content %}...{% endblock %}`.
- Access config values with `{{ config.title }}`, `{{ config.extra.author }}`, etc.
- Access page values with `{{ page.title }}`, `{{ page.date }}`, `{{ page.description }}`.
- Loop over posts: `{% for page in section.pages %}`.
- Filters: `| date(format="%B %d, %Y")`, `| safe`, `| truncate(length=200)`.
- Whitespace control: use `{%- -%}` to strip surrounding whitespace.

### HTML Conventions

- Use semantic elements: `<article>`, `<header>`, `<nav>`, `<main>`, `<footer>`, `<figure>`, `<figcaption>`.
- Every navigational landmark gets an `aria-label`.
- Active nav links use `aria-current="page"`.
- The skip link (`<a href="#main" class="skip-link">`) must stay in `base.html`.
- Heading hierarchy must be correct (`h1` once per page, then `h2`, `h3`...).
- **No inline `style=` attributes** except what Zola's syntax highlighter emits.
- **No `<script>` tags** anywhere.

---

## CSS Conventions

All styling lives in a single file: `static/style.css`.

- Uses **CSS custom properties** (`--color-bg`, `--color-text`, `--color-accent`, etc.) defined in `:root`.
- Dark mode via `@media (prefers-color-scheme: dark)` — override variables there.
- High-contrast mode via `@media (forced-colors: active)`.
- Layout uses CSS Grid for the two-column (text | images) wide-screen layout.
- **No CSS preprocessors** (Sass is disabled: `compile_sass = false` in config.toml).
- **No external fonts** — system fonts only: `Georgia, serif` for body, `"Courier New", monospace` for code.
- Do not split CSS into multiple files. Keep it in one place.

---

## Security Headers

`static/_headers` defines Cloudflare Pages rules. The current policy is intentionally tight:

```
Content-Security-Policy: default-src 'none'; style-src 'self' 'unsafe-inline'; img-src 'self'; font-src 'self'; base-uri 'self'; form-action 'none'; frame-ancestors 'none'
```

The `'unsafe-inline'` on `style-src` is required because Zola's syntax
highlighter emits inline `style=` attributes on `<span>` elements. If you
switch to class-based highlighting (`highlight_themes_css` in config.toml),
you can tighten this to `style-src 'self'` only.

**Do not add new directives that permit external origins** (e.g., no `script-src cdn.example.com`).

---

## Deployment

Deployment is automatic via Cloudflare Pages:

1. Push to GitHub main branch.
2. Cloudflare detects the push, runs `zola build`, serves the `public/` directory.
3. `_headers` and `_redirects` are processed natively by Cloudflare (no plugins needed).

Cloudflare Pages environment variable required:
```
ZOLA_VERSION = 0.19.2
```

**Do not** add any server-side code, API routes, or Cloudflare Workers unless
explicitly requested — the site is intentionally static.

---

## Key Design Decisions (Do Not Reverse Without Discussion)

| Decision | Reason |
|---|---|
| Zero JavaScript | Privacy, performance, accessibility, no attack surface |
| Single CSS file | One HTTP request, no render-blocking, easy to reason about |
| No external CDN assets | No third-party tracking, tight CSP, offline-friendly |
| WebP + JPEG pairs | Modern browser support with graceful fallback |
| System fonts only | Instant render, no FOUT, no font-loading complexity |
| `minify_html = true` | Reduces transfer size with zero effort |
| HSTS + preload | Forces HTTPS site-wide including subdomains |

---

## What AI Assistants Should and Should Not Do

### Do
- Edit Markdown files in `content/` to add or update posts.
- Add new templates in `templates/` following existing patterns.
- Extend `static/style.css` using existing CSS custom properties.
- Update `static/llms.txt` when new posts are added.
- Add URL redirects to `static/_redirects` when slugs change.
- Run `zola build` to verify the site builds without errors before committing.

### Do Not
- Add `<script>` tags or JavaScript of any kind.
- Add external stylesheet or font links.
- Create new files not needed by Zola (no `package.json`, no `.github/` CI, etc.).
- Loosen the Content-Security-Policy to allow external origins.
- Add server-side logic, API endpoints, or dynamic features.
- Use inline `style=` attributes (except what the syntax highlighter emits).
- Add raw `<img>` tags in content — always use the `{{ img(...) }}` shortcode.
- Introduce Sass or any CSS preprocessor.

---

## Zola-Specific Notes

- `config.toml` is the single source of truth for site-wide settings.
- `config.extra.*` fields are available in all templates as `{{ config.extra.field }}`.
- The `now()` function in templates returns the build time (used in footer copyright year).
- Zola auto-generates `/sitemap.xml`, `/rss.xml`, and `/tags/` pages from content + config.
- Pages with `draft = true` in frontmatter are excluded from production builds.
- The `get_url(path="...")` Tera function generates absolute URLs using `base_url` from config.
- Asset fingerprinting is built in — Zola hashes filenames so aggressive caching is safe.

---

## Author

**Mara Vexa** — security engineer, technical writer.
- Site: [maravexa.com](https://maravexa.com)
- GitHub: [github.com/maravexa](https://github.com/maravexa)
- Email: me@maravexa.com
