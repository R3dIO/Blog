---
title: "A Draft Horse Can't Plow a Field Without a Harness. Neither Can Your AI Coding Tool."
date: 2026-09-07
tags: [ai-agents, claude-code, github-copilot, google-antigravity, agentic-coding, devops, sre, platform-engineering]
permalink: /posts/ai-harness-for-devops-sre/
---

Put the strongest draft horse in the world in an open field and it will do exactly nothing useful. Not because it lacks strength — strength was never the constraint. What turns that horse into a plow, a cart, or a rescue team is the harness: the straps, the buckles, the exact points where its power is allowed to attach to the world. Take the harness off a smart animal and you get wandering. Put a sloppy harness on a smart animal and you get it dragging the wrong thing, or hurting itself against a fence post it never saw coming.

That's not a metaphor I'm stretching for effect — it's literally the term the industry uses. Every Copilot, Claude Code, or Google Antigravity session runs inside something practitioners now call an "agent harness": the execution loop, the tool permissions, the memory, the guardrails wrapped around the model. And the uncomfortable finding coming out of teams running these tools in production is that swapping which model you have — this one instead of that one — moves the needle far less than most people assume. What moves it is the harness.

For a DevOps or SRE engineer, this isn't academic. You're not asking an AI to draft an email — you're pointing it at Terraform, Kubernetes manifests, incident runbooks, and production access. A badly harnessed agent in that context doesn't just produce mediocre output; it produces mediocre output with write access to things that page people at 2am. So before the "which tool is best, Copilot or Claude Code or Antigravity" debate, there's a more useful question: what does a good harness actually look like, and how do you build one for infrastructure work specifically?

## Where the harness actually lives

Strip away the marketing and every one of these tools is built from the same stack: a model at the bottom, a harness wrapped around it, sometimes a framework on top for multi-agent orchestration, and a platform that ships the whole thing to you as a product. The model is the part everyone argues about. The harness is the part that actually decides whether a session goes well, and it comes down to a handful of concrete pieces:

**What it can see.** Live repository context, your runbooks, past incident write-ups — not just the ticket you pasted in. An agent reasoning from two sentences and no context behaves like a new hire handed a Jira ticket with no access to the wiki.

**What it's allowed to touch.** Predefined tools with real validation behind them, not "can run any shell command." That's the difference between an agent that can read your cluster state and one that can also silently apply changes to it.

**What it remembers.** Session memory that survives past a single conversation, so you're not re-explaining your Terraform module layout every Monday morning.

**How much it's allowed to spiral.** Context clipping and bounded subagents — child tasks with their own permission ceiling and their own recursion limit, so a debugging detour doesn't quietly turn into the agent reading your entire codebase twice.

Different tools implement this differently. **Claude Code** leans on a `CLAUDE.md` memory file, hooks that intercept tool calls before they run, and subagents you can route to cheaper or more capable models depending on the task. **GitHub Copilot** has moved the same idea into `copilot-instructions.md` plus an agent mode that can plan and execute multi-file changes, sitting alongside completions that were never metered or gated at all. **Google Antigravity**, which shipped its 2.0 desktop app and CLI at I/O 2026, takes it further: it spans a desktop app, CLI, SDK, and IDE extension sharing one harness, with declarative approval policies and "visual artifacts" — recorded plans, diffs, browser actions — so a human can audit what an autonomous session actually did without re-running it.

None of this is theoretical fine print. One team documented raising their agent's task success rate from 80% to 100% not by switching models, but by removing 80% of the tools it had access to. Less rope, better aim. For infrastructure work, where a wrong tool call can mean a cluster instead of a file, that finding should reframe how you think about setup entirely.

## Why this matters beyond engineering

A few things make this relevant even if you're not the one writing prompts:

**The productivity story is real, but it's concentrated in the people who set this up well.** Software developer job postings are up roughly 15% since agentic coding tools went mainstream, even as overall postings fell — but the bulk of that rebound sits in senior, AI-fluent roles, not entry-level ones. The gap between a team getting real leverage out of these tools and one getting expensive noise usually isn't the subscription tier. It's whether anyone configured the harness.

**It's a governance question before it's a productivity one.** An agent with unscoped access to your infrastructure is a new hire you've handed prod credentials to on day one, with no onboarding. Most engineering orgs would never do that with a person. The bar shouldn't be lower because the "new hire" happens to be software.

**It's becoming a hiring signal in both directions.** LinkedIn's own 2026 data lists Cloud Engineers, Site Reliability Engineers, and Platform Engineers among its fastest-growing roles, with prompt engineering and AI literacy named as skills on the rise across the board. Candidates who can talk about *how* they set up and scope an agent — not just that they use one — are increasingly the ones standing out.

**It sits right next to the cost conversation your team is probably already having.** A well-harnessed agent is also a cheaper one — tighter context, fewer wasted tool calls, less runaway looping. If your team hasn't looked at that side of it yet, it's worth pairing with [how these tools actually bill](/posts/ai-coding-cost-optimization/).

## For the engineers reading this

Here's what the setup actually looks like in practice, tool by tool.

**Across all of them, before you touch a specific tool:** scope permissions the way you'd scope an IAM role — read-only by default, write access earned per task, prod always behind a human gate. Keep a project-level instructions file in the repo instead of leaving it to memory or habit, and treat that file as part of the codebase: reviewed, versioned, updated when the stack changes.

**Claude Code:** keep `CLAUDE.md` lean and let skills load workflow-specific detail only when invoked. Use `PreToolUse` hooks to filter noisy command output — test runs, log dumps — down to the actual failure before it reaches the model's context. Route by task difficulty: your default model for most infra work, the top-tier model reserved for genuinely hard multi-step reasoning, a small model for simple subagent chores. Turn on plan mode before any change touching more than one file, so exploration happens once instead of three times after a wrong turn.

**GitHub Copilot:** lean on completions for everyday code — they're unmetered and don't need supervision. Reach for agent mode specifically for multi-file, well-scoped tasks (a Terraform module refactor, a pipeline update), and write `copilot-instructions.md` the way you'd write onboarding notes for a contractor: exact build commands, exact test commands, the things they can't infer from the README.

**Google Antigravity:** use its parallel subagents for genuinely background work — a dependency audit, a log sweep — while you stay on the primary task, and actually read the visual artifacts before approving a plan rather than rubber-stamping it because the diff looks plausible. Tool-approval gates are only as good as whether someone bothers to look at what they're gating.

**Running more than one of these side by side?** Don't maintain three separate instruction files that drift out of sync. `AGENTS.md` has emerged as the closest thing to a cross-tool standard — Copilot, Cursor, OpenAI Codex, and a growing list of others read it natively. Claude Code is the one holdout that reads `CLAUDE.md` by default, but a one-line `@AGENTS.md` import at the top of that file solves it. Write the setup once.

## Scaling this across a team

The tactics above are for one engineer's setup. At team scale, the job shifts from "configure your own harness well" to "make it hard to configure one badly."

**Commit the harness config, don't leave it to preference.** `CLAUDE.md`, `AGENTS.md`, `copilot-instructions.md` — these belong in version control next to your linting rules, reviewed the same way, updated when the stack changes rather than going stale the week someone wrote them.

**Treat agent permissions like any other access request.** Least privilege by default, elevated write scope reviewed and time-boxed the same way you'd review a person's request for prod access — because functionally, that's what it is.

**Keep the human checkpoint regardless of which tool wrote the change.** A PR review and a staging gate don't get skipped because the diff came from an agent instead of a person. If anything, that's the one control you loosen last, after everything above is already dialed in.

**Start measuring which agent authored what.** Most teams already have an incident postmortem culture; extend it. If a change causes a problem, "which harness configuration produced this and what did it have access to" should be answerable in minutes, not a mystery.

## The short version

Right now, the temptation is to treat "which AI coding tool should my team use" as the whole decision. It isn't. The model you pick matters less than most vendors would like you to believe — the harness around it is what decides whether a session produces something useful or something that needs cleaning up afterward. For DevOps and SRE work specifically, that harness is really an onboarding process wearing a different name: give the agent context, scope its access, give it memory, and keep a human at the checkpoint. Get that right and it stops mattering much whether the name on the tool is Copilot, Claude Code, or Antigravity.

If your team is figuring out how to roll agentic tooling into infrastructure work without turning it into a new class of incident — or you're hiring for a role where this kind of thinking matters — I'm happy to talk through it in plain terms. Reach out any time.

Sources:
- [A Comparison of AI Agent Harnesses in 2026](https://winder.ai/ai-agent-harness-comparison/)
- [Google Antigravity Docs](https://antigravity.google/docs/home/)
- [Google launches Antigravity 2.0 with an updated desktop app and CLI tool at I/O 2026 | TechCrunch](https://techcrunch.com/2026/05/19/google-launches-antigravity-2-0-with-an-updated-desktop-app-and-cli-tool-at-io-2026/)
- [Top 7 AI Tools Every DevOps and SRE Engineer Needs in 2026](https://dev.to/meena_nukala/top-7-ai-tools-every-devops-and-sre-engineer-needs-in-2026-242c)
- [AGENTS.md Spec Guide](https://www.morphllm.com/agents-md-guide)
- [AI and job postings: from destruction to creation | Indeed Hiring Lab](https://www.hiringlab.org/2026/07/08/ai-and-job-postings-from-destruction-to-creation/)
- [LinkedIn Jobs on the Rise 2026](https://www.dice.com/career-advice/ai-related-jobs-top-linkedins-fastest-growing-roles-list-for-2026)
- [LinkedIn Skills on the Rise 2026](https://news.linkedin.com/2026/Skills-on-the-rise-2026)
