# Fiky's Blog

Personal blog built with [Astro](https://astro.build), deployed to GitHub Pages.

Live at [porto.fikyb.my.id](https://porto.fikyb.my.id)

## Stack

- **Astro 5** — static site generator
- **Markdown** — blog posts in `posts/`
- **GitHub Pages** — hosting (built to `docs/`)

## Commands

| Command           | Action                                      |
| :---------------- | :------------------------------------------ |
| `npm install`     | Install dependencies                        |
| `npm run dev`     | Start local dev server at `localhost:4321`   |
| `npm run build`   | Build production site to `./docs/`          |
| `npm run preview` | Preview build locally before deploying      |

## Adding a Post

Create a `.md` file in `posts/` with frontmatter:

```markdown
---
title: My New Post
slug: my-new-post
description: Short description here
tags:
  - technical
added: 2026-07-21T00:00:00.000Z
---

Post content here.
```

Then `npm run build` and push.

## Credits

Template based on [blahg](https://github.com/cassidoo/blahg) by [Cassidy Williams](https://github.com/cassidoo). Thank you for making such a clean and simple blog template!
