---
layout: post
title: getting this site live on GitHub Pages
date: 2026-07-09 06:00:00
description: how pwnkhat.github.io went from a private repo to a working site, and the two silent bugs that almost stopped it
tags: meta github-pages jekyll
categories: sample-posts
---

I finally got this site live at `pwnkhat.github.io`. It went smoother than expected, but there were two quiet bugs along the way that are worth writing down — mostly so future-me doesn't have to debug them twice.

#### The starting point

The repo already had everything wired up: it was pushed to GitHub, and a GitHub Actions workflow was already set up to build the Jekyll site and deploy it to a `gh-pages` branch on every push to `main`. So the "push code to GitHub" part was already done.

What wasn't done was **GitHub Pages itself**. Even with a repo named `<username>.github.io`, GitHub doesn't automatically turn Pages on — you still have to go to the repo's **Settings → Pages** and pick a source branch. Until that happens, the domain just 404s, no matter how good the build is sitting in `gh-pages`.

#### Bug 1: leftover template config

While digging into the 404, I found `_config.yml` still had the stock template defaults:

- `url` was set to the original theme author's domain, not mine
- `baseurl` was set to `/al-folio` — a subpath, which is wrong for a user page that needs to serve from the domain root

Both got fixed to point at the real domain with an empty `baseurl`.

#### Bug 2: GitHub re-running Jekyll on an already-built site

This one was sneakier. The deploy workflow builds the site and pushes the finished HTML/CSS/JS to `gh-pages` — but without a `.nojekyll` file in that output, GitHub Pages runs **its own** Jekyll pass over it before serving. That second pass quietly strips anything sitting in an underscore-prefixed folder and can otherwise mangle a site that's already built.

The fix was one line added to the deploy workflow, right before the deploy step:

```yaml
- name: Disable Jekyll processing on GitHub Pages 🚫
  run: touch _site/.nojekyll
```

#### "It's live but there's no styling"

After turning Pages on, the site loaded but completely unstyled — just text and images. Checking the response headers on the live CSS files showed everything was actually fine: correct content type, correct content, and timestamps matching the fixed deploy. The unstyled version was just the browser showing a cached copy of the *previous*, broken deploy. A hard refresh cleared it up.

#### What's left

Hosting is done — the site rebuilds and redeploys automatically on every push now. What's left is just filling in real content: name, bio, profile photo, publications, projects, and so on, since it's all still the theme's placeholder text for now. That's the fun part, though.
