# Blog

Blog for everything related to Cloud/Platform Eng. and DevOps.

This repo is my running knowledge base — each idea lives as a `README.md` in
its own folder under `posts/`. It's published automatically via GitHub Pages
at **[blog.akoli.dev](https://blog.akoli.dev)**.

## Adding a new post

1. Copy `POST_TEMPLATE.md` into a new folder:

   ```
   posts/YYYY-MM-DD-your-slug/README.md
   ```

2. Fill in the front matter (`title`, `date`, `tags`, `permalink`) and write
   the post below it.
3. Commit and push to `main` — GitHub Pages rebuilds the site automatically.

## Previewing locally (optional)

```
bundle install
bundle exec jekyll serve
```

Then open `http://localhost:4000`.

## Structure

```
.
├── _config.yml          # Jekyll site config
├── index.md              # Homepage — lists all posts under posts/
├── POST_TEMPLATE.md      # Copy this to start a new post
├── CNAME                 # Custom domain: blog.akoli.dev
└── posts/
    └── YYYY-MM-DD-slug/
        └── README.md      # One folder per idea
```
