---
title: "A Good Kitchen Never Re-Preps a Dish It Already Made. Most Kubernetes Clusters Serving LLMs Do — Until llm-d."
date: 2026-09-13
tags: [llm-d, kserve, vllm, kubernetes, ai-infrastructure, mlops, platform-engineering, cncf, open-source, generative-ai]
permalink: /posts/llm-d-kubernetes-inference/
---

A good restaurant kitchen never makes a customer wait for a dish it's already half-made. Order the same curry as the table next to you ten minutes ago, and a good cook doesn't start from a bare pot — the base is already simmering, the vegetables are already chopped, and it lands on your table in a fraction of the time. Now picture the same kitchen with one rule changed: every order gets handed to whichever cook happens to be free, with no memory of who already prepped what, and no distinction between the cook doing the chopping and the cook doing the plating. That kitchen would burn through ingredients, cooks, and time re-doing work it finished an hour ago.

That second kitchen is roughly how most Kubernetes clusters have been serving large language models. Every request — no matter how similar to the one before it — gets routed to whatever GPU replica is next in line, with no awareness that another replica already has most of the relevant work sitting in memory. It works. It's also why AI inference bills read like a kitchen that re-chops the same onions all day.

There are actually two very different jobs bundled into every LLM response, and treating them as one job is part of the waste. Reading and understanding your prompt — call it the prep work — is compute-heavy and happens once. Generating the response, one word at a time — call it the cooking — is a different kind of work entirely: it repeats constantly, and it lives or dies on how fast the system can reach back into what it already knows about your specific conversation (technically, its "KV cache"). A cluster that treats prep and cooking as identical, interchangeable work, and forgets who's already prepped what, is leaving real money and real speed on the table.

That's the exact problem **llm-d** — a Kubernetes-native inference project backed by Red Hat, Google, IBM Research, NVIDIA, and CoreWeave — was built to fix. Paired with **KServe**, the project that already handles the "restaurant management" side of model serving on Kubernetes, you get something close to a fully staffed, well-run kitchen instead of a room full of cooks working from instinct.

## Where the waste actually happens

Every time a large language model answers you, it moves through two distinct phases. **Prefill** is the model reading your entire prompt at once — a burst of heavy computation that scales with how much text you handed it. **Decode** is the model writing its answer back to you, one token at a time, over and over — slower, steadier work that depends on fast access to everything the model has already worked out about that conversation.

Most Kubernetes setups treat every LLM-serving pod as an identical, interchangeable box and hand out requests round-robin — the way a call center might route calls without checking whether the agent who just spoke to this customer is still free. Two problems follow from that. First, prefill and decode get crammed onto the same hardware even though they want opposite things — prefill wants raw compute, decode wants fast memory access — so you end up over-provisioning both to cover the other's needs. Second, when a new request is a near-duplicate of one just handled (the same system prompt, the same few-shot examples, a long conversation continuing), a round-robin router has no way of knowing that, and sends it to a replica that has to redo work another replica already finished.

## Why this matters beyond engineering

**It's a GPU bill, and GPUs are the most expensive line item in most AI budgets.** Re-doing prefill work that's already cached, and running two workloads with opposite hardware needs on identical machines, means paying for more capacity than the traffic actually requires. Google's own early testing showed roughly a 2x improvement in how fast a model produces its first response token for code-completion workloads once cache-aware routing and disaggregation were in place — the same requests, answered faster, on the same hardware.

**It's not a bet on an unproven side project.** llm-d launched in May 2025 out of Red Hat, Google, and IBM Research, with NVIDIA and CoreWeave as founding partners, and has since picked up AMD, Cisco, Hugging Face, Intel, Lambda, and Mistral AI, plus research backing from UC Berkeley and the University of Chicago. In March 2026 it joined the Cloud Native Computing Foundation as a Sandbox project — the same governance home Kubernetes and KEDA came up through — meaning its direction is set by a broad community rather than one vendor's roadmap. It's early-stage (Sandbox is CNCF's earliest tier), but it's early-stage with unusually wide industry buy-in.

**It hedges against hardware lock-in.** llm-d is explicitly built to work across GPUs and TPUs and across inference engines, which matters for any company trying to avoid getting boxed into one accelerator vendor while chip supply and pricing keep shifting.

**It's becoming a hiring signal.** The same 2026 data showing Cloud Engineers, SREs, and Platform Engineers among LinkedIn's fastest-growing roles also points to AI infrastructure literacy as one of the differentiating skills inside those roles. Understanding how a company actually serves its models in production — not just which model it picked — is quickly becoming part of that bar.

## For the engineers reading this

Here's how the pieces actually fit together, since "llm-d" and "KServe" solve different problems and get confused as competitors more often than they should.

**KServe is the control plane.** It's the layer that already handles model lifecycle on Kubernetes: the `LLMInferenceService` custom resource gives you an OpenAI-compatible endpoint (`/v1/chat/completions`, streaming included) in front of a runtime like vLLM or Hugging Face TGI, wired into the Kubernetes Gateway API, with autoscaling that can scale a model all the way to zero when nothing's using it. If you're already running KServe for predictive or generative models, none of that goes away.

**llm-d is the traffic intelligence sitting inside that control plane.** What KServe doesn't do on its own is look across a fleet of replicas and make routing decisions based on which one already has your prompt's prefix cached, which one is closer to its memory limit, or which requests should go to prefill-optimized hardware versus decode-optimized hardware. That's llm-d's job, implemented through the Gateway API Inference Extension and its Endpoint Picker Protocol — a pluggable scheduler that scores each candidate replica on cache residency, current queue depth, GPU utilization, and SLA constraints before picking where a request goes.

**Disaggregation** means literally running prefill and decode as separate pools of pods — sometimes on different GPU generations — so each phase gets sized for what it actually needs instead of both being squeezed onto identical, one-size-fits-all replicas.

**Multi-tier KV cache management** extends that same logic to memory: hot prefixes stay in GPU memory, less active ones get offloaded to CPU memory or disk instead of being evicted and recomputed from scratch — which is where a lot of the practical throughput gains show up on long or branching conversations.

If you're deciding where to start: teams already running KServe can adopt `LLMInferenceService` first and layer in llm-d's scheduler for the caching and disaggregation wins, without re-architecting the serving layer. Teams starting from scratch can go straight to llm-d's own quickstart, which ships opinionated "well-lit path" configurations rather than making you assemble the Inference Gateway, vLLM, and scheduler wiring by hand. Either way, it's worth remembering this is a CNCF Sandbox project as of March 2026 — young enough that APIs can still shift — so pilot it on a workload you can afford to babysit before it becomes your default serving path.

## The short version

Kubernetes solved "how do I run a lot of identical, stateless pods" years ago. LLM inference isn't that problem — every request carries state in the form of its cache, and every response is really two different workloads wearing one trenchcoat. KServe still gets you a governed, autoscaled, OpenAI-compatible way to serve a model; llm-d is what makes the traffic between those replicas actually smart about the caching and hardware-shape problem underneath it. Together, they're the closest thing Kubernetes has right now to that well-run kitchen — one that remembers who already has your order half-made instead of starting from scratch every time.

If your team is looking at GPU spend on inference and wondering where it's actually going, or hiring for a role where this kind of systems thinking matters — I'm happy to talk through it in plain terms. Reach out any time.

Sources:
- [Announcing the llm-d community](https://llm-d.ai/blog/llm-d-announce)
- [Enhancing vLLM for distributed inference with llm-d](https://cloud.google.com/blog/products/ai-machine-learning/enhancing-vllm-for-distributed-inference-with-llm-d) — Google Cloud Blog
- [llm-d: Kubernetes-native distributed inferencing](https://developers.redhat.com/articles/2025/05/20/llm-d-kubernetes-native-distributed-inferencing) — Red Hat Developer
- [Welcome llm-d to the CNCF: Evolving Kubernetes into SOTA AI infrastructure](https://www.cncf.io/blog/2026/03/24/welcome-llm-d-to-the-cncf-evolving-kubernetes-into-sota-ai-infrastructure/) — CNCF
- [llm-d officially a CNCF Sandbox project](https://cloud.google.com/blog/products/containers-kubernetes/llm-d-officially-a-cncf-sandbox-project) — Google Cloud Blog
- [Best of Both Worlds: Cloud-Native AI Inference at Scale using KServe and llm-d](https://kserve.github.io/website/blog/cloud-native-ai-inference-kserve-llm-d) — KServe
- [Combining KServe and llm-d for optimized generative AI inference](https://developers.redhat.com/articles/2026/04/21/kserve-llm-d-optimized-gen-ai-inference) — Red Hat Developer
- [Understanding LLMInferenceService](https://kserve.github.io/website/docs/model-serving/generative-inference/llmisvc/llmisvc-overview) — KServe
- [LinkedIn Skills on the Rise 2026](https://news.linkedin.com/2026/Skills-on-the-rise-2026)
