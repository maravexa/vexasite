# maravexa.us

Personal website built with [Zola](https://www.getzola.org/).

## Prerequisites

Install Zola (single binary, no other dependencies):

```bash
# macOS
brew install zola

# Arch Linux
pacman -S zola

# Ubuntu
snap install zola --edge

# Or download a binary from https://github.com/getzola/zola/releases
```

## Local development

```bash
zola serve
# → opens at http://127.0.0.1:1111
# → live-reloads on file changes
```

## Build

```bash
zola build
# → output in /public
```

## Adding a blog post

Create a new Markdown file in `content/blog/`:

```
content/blog/my-new-post.md
```

Frontmatter:

```toml
+++
title = "My New Post"
date = 2025-03-15
description = "A one-sentence summary for the post list and SEO."
[taxonomies]
tags = ["tag1", "tag2"]
+++

Your content here.
```

## Adding images to a post

1. Put your image in `static/img/` — provide both `.webp` and `.jpg`
   versions for best browser compatibility. Aim for <150 KB per image.

   Quick conversion with cwebp:
   ```bash
   cwebp -q 82 photo.jpg -o photo.webp
   ```

2. Reference in your post with the shortcode:
   ```
   {{ img(src="photo.webp", alt="Describe the image", caption="Optional caption", w=800, h=533) }}
   ```

   The `w` and `h` attributes prevent layout shift (CLS). Always include them.

## Deployment (Cloudflare Pages)

1. Push this repo to GitHub.
2. In Cloudflare Pages, create a new project → connect your repo.
3. Build settings:
   - Framework preset: None
   - Build command: `zola build`
   - Build output directory: `public`
   - Environment variable: `ZOLA_VERSION` = `0.19.2` (or latest)
4. Deploy. Cloudflare handles HTTPS, CDN, and the `_headers` / `_redirects` files automatically.

## File structure

```
├── config.toml              # Site configuration
├── content/
│   ├── _index.md            # Homepage content
│   ├── about.md             # About page
│   └── blog/
│       ├── _index.md        # Blog section settings
│       └── *.md             # Posts (one file per post)
├── static/
│   ├── style.css            # Entire stylesheet (~5KB)
│   ├── robots.txt
│   ├── _headers             # Cloudflare cache rules
│   ├── _redirects
│   └── img/                 # Images (.webp + .jpg)
└── templates/
    ├── base.html            # Shared HTML shell
    ├── index.html           # Homepage
    ├── page.html            # Generic page (About, etc.)
    ├── blog-list.html       # Post list
    ├── blog-page.html       # Single post
    └── shortcodes/
        └── img.html         # {{ img(...) }} shortcode
```
