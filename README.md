# lvcc2018.github.io

Personal website & blog — built with [Hugo](https://gohugo.io), deployed via GitHub Pages.

## Setup

```bash
# Install Hugo
brew install hugo

# Run locally
hugo server -D
```

## Write a post

```bash
hugo new content blog/YYYY-MM-DD-title.md
```

Write in Markdown. Push to `main` → GitHub Actions builds & deploys automatically.

## Structure

```
content/
├── _index.md        ← Homepage
├── about/_index.md  ← Resume / About
└── blog/            ← Technical blog posts
```
