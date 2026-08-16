---
title: "Why I'm Building This Knowledge Base in the Open"
date: 2026-08-16
tags: [cloud, platform-engineering, devops, knowledge-management]
permalink: /posts/why-im-building-this-in-the-open/
---

Most engineers keep two notebooks. One is the polished kind — the blog post, the conference talk, the LinkedIn update — written after the dust has settled and the idea already works. The other is the messy one: the scratch notes, the half-formed diagrams, the "why does this keep breaking" moments that actually taught you something. Almost nobody publishes the second notebook. I want to.

This repo is that notebook. Every idea I add here is a folder with a single `README.md` in it — nothing fancier than that. No CMS, no database, no build pipeline to maintain. Just Markdown files living in Git, the same way the code I write during the day already lives. If an idea is worth keeping, it gets a folder. If it's worth sharing, GitHub Pages turns it into a page at [blog.akoli.dev](https://blog.akoli.dev) automatically the moment I push.

Think of it like a lab notebook versus a published paper. A published paper shows you the conclusion. A lab notebook shows you the *thinking* — the dead ends, the "wait, that's interesting," the moment a messy workaround turned into a pattern worth reusing. Cloud and platform engineering is full of that kind of thinking: the tradeoffs between a managed service and something you run yourself, the incident that taught you more about your system than the architecture diagram did, the small script that quietly saves your team an hour a week. That's the stuff I want to capture here — not because it's polished, but because it's real.

A few things worth knowing about how this works, in case you want to build something similar:

**The structure is deliberately boring.** Each idea lives at `posts/YYYY-MM-DD-slug/README.md`. Boring is a feature — it means I can add a new idea in the time it takes to create a folder and start typing, without deciding on tooling every time.

**Git is the whole publishing pipeline.** There's no separate "publish" step. Writing the note and shipping it are the same action: `git push`. That removes almost every excuse I've ever had for not writing something down.

**It's a knowledge base first, a blog second.** The blog view at blog.akoli.dev is just one way of reading this content. The real value is the growing folder of ideas underneath it — searchable, versioned, and linkable, the same way I'd want my own reference material to be.

If you're the kind of person who takes notes on how systems actually behave in production — not just how the docs say they should — I hope some of what ends up here is useful to you too. More to come.
