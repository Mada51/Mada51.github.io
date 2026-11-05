# mada51.github.io

Jekyll 4 site (theme: minima), deployed to GitHub Pages by GitHub Actions.

## Local development

```bash
bundle install
bundle exec jekyll serve --livereload   # http://127.0.0.1:4000
```

Build only:

```bash
bundle exec jekyll build                # output in _site/
```

## Deployment

Pushing to `main` runs `.github/workflows/pages.yml`, which builds the site and
deploys it. One-time setting: **Settings → Pages → Build and deployment →
Source = GitHub Actions**.

## Layout

| Path | Purpose |
|---|---|
| `_config.yml` | Site settings, plugins |
| `_posts/` | Blog posts (`YYYY-MM-DD-title.md`) |
| `index.md`, `about.md`, `404.html` | Pages |
| `assets/` | Static files (images, CSS overrides) |
