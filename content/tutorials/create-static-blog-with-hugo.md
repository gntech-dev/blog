---
title: "How to Create a Static Blog with Hugo"
date: 2026-05-01
categories: [Hugo, Static Site, Tutorial]
tags: [hugo, static-site, blog, cloudflare, github, markdown]
author: gnolasco
---

Hugo is a fast static site generator that lets you build a blog using plain Markdown files. It is a good fit for technical blogs because it is simple to maintain, easy to version with Git, and produces static HTML that can be hosted almost anywhere.

In this guide, we will create a basic Hugo blog, add a theme, write a first post, build the site, and prepare it for deployment.

## Why Use Hugo

Hugo is useful when you want a blog that is:

- Fast to load
- Easy to host
- Simple to back up with Git
- Written in Markdown
- Low maintenance
- Independent from databases and server-side runtimes

A Hugo blog can be hosted on platforms like Cloudflare Pages, GitHub Pages, Netlify, a VPS, an object storage bucket, or even a Cloudflare Worker setup.

## Prerequisites

You will need:

- A computer with Linux, macOS, or Windows
- Git installed
- Hugo installed
- Basic terminal knowledge

Check if Hugo is installed:

```bash
hugo version
```

If Hugo is not installed, follow the official installation guide:

```text
https://gohugo.io/installation/
```

## 1. Create a New Hugo Site

Create a new site directory:

```bash
hugo new site my-blog
cd my-blog
```

This creates the basic Hugo structure:

```text
my-blog/
├── archetypes/
├── assets/
├── content/
├── data/
├── hugo.toml
├── i18n/
├── layouts/
├── static/
└── themes/
```

The most important folders at the beginning are:

- `content/` — your Markdown posts and pages
- `static/` — static files like images, icons, and robots.txt
- `themes/` — your Hugo theme
- `hugo.toml` — your site configuration

## 2. Add a Theme

For a clean blog, PaperMod is a popular Hugo theme.

Add it as a Git submodule:

```bash
git init
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

Then edit `hugo.toml`:

```toml
baseURL = 'https://example.com/'
languageCode = 'en-us'
title = 'My Blog'
theme = 'PaperMod'
```

Replace `https://example.com/` with your real domain when you are ready to publish.

## 3. Configure Basic Blog Metadata

Add useful metadata to `hugo.toml`:

```toml
[params]
description = "Technical notes, tutorials, and personal blog posts."
images = ["/images/blog-cover.png"]

[params.homeInfoParams]
Title = "My Blog"
Content = "Technical notes, tutorials, and practical guides."
```

The `description` helps search engines and social previews understand your site. The `images` value is used for Open Graph previews when links are shared.

## 4. Create Your First Post

Create a new post:

```bash
hugo new posts/my-first-post.md
```

Open the file under `content/posts/my-first-post.md` and update it:

```markdown
---
title: "My First Post"
date: 2026-05-01
draft: false
tags: [hugo, blog]
categories: [Tutorial]
---

Welcome to my new Hugo blog.

This post was written in Markdown and generated as a static HTML page.
```

Make sure `draft: false` is set if you want the post to appear in production builds.

## 5. Run the Local Development Server

Start Hugo locally:

```bash
hugo server -D
```

Then open:

```text
http://localhost:1313
```

The `-D` flag includes draft posts while previewing locally.

## 6. Build the Static Site

Build the production version:

```bash
hugo --minify
```

Hugo will generate the static site inside the `public/` directory:

```text
public/
├── index.html
├── posts/
├── sitemap.xml
├── index.xml
└── assets/
```

The `public/` folder is what gets deployed to your hosting provider.

## 7. Add robots.txt

Create `static/robots.txt`:

```txt
User-agent: *
Allow: /

Sitemap: https://example.com/sitemap.xml
```

When Hugo builds the site, this file will be copied to:

```text
/public/robots.txt
```

## 8. Deploy the Blog

There are many ways to deploy Hugo. A simple Git-based flow looks like this:

```text
Write Markdown → Commit to GitHub → Build Hugo → Publish public/ directory
```

Common deployment options include:

- Cloudflare Pages
- GitHub Pages
- Netlify
- A static file server with Nginx or Caddy
- Cloudflare Worker serving built files

If you already use Cloudflare, Cloudflare Pages is one of the easiest options because it can build Hugo directly from your GitHub repository.

Example Cloudflare Pages settings:

```text
Framework preset: Hugo
Build command: hugo --minify
Build output directory: public
```

## 9. Recommended Blog Structure

For a technical blog, a clean structure is helpful:

```text
content/
├── about.md
├── posts/
│   └── my-first-post.md
└── tutorials/
    └── linux-server-hardening.md
```

You can use `posts/` for general writing and `tutorials/` for technical guides.

## 10. Maintain the Blog

A good workflow is:

```bash
git pull
hugo new tutorials/new-guide.md
hugo server -D
hugo --minify
git add .
git commit -m "Add new Hugo tutorial"
git push
```

If your hosting provider is connected to GitHub, the push can automatically trigger a rebuild and publish the new version.

## Conclusion

Hugo is a solid choice for a fast, reliable static blog. You write content in Markdown, keep everything in Git, and deploy static files without needing a database or backend service.

For a homelab or infrastructure-focused blog, Hugo provides a practical balance of speed, simplicity, and long-term maintainability.
