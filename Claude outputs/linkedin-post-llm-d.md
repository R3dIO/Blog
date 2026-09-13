A good kitchen never makes you wait for a dish it's already half-made. Order the same thing as the table next to you, and a good cook reaches for what's already prepped instead of starting from a bare pot.

Most Kubernetes clusters serving LLMs don't work that way. Every request gets routed round-robin to whatever GPU is free, with zero memory of which replica already has your conversation's cache sitting right there in memory. That's not a small inefficiency — it's the difference between a kitchen that reuses its prep work and one that re-chops the same onions all day.

This is exactly what llm-d — the Kubernetes-native inference project backed by Red Hat, Google, IBM Research, NVIDIA, and now a CNCF Sandbox project — was built to fix. Paired with KServe, which already handles model lifecycle, autoscaling, and OpenAI-compatible endpoints, you get cache-aware routing and prefill/decode disaggregation: the closest thing Kubernetes has right now to a well-run kitchen instead of a room full of cooks working from instinct.

I wrote up the full breakdown on my blog — one section for anyone deciding whether this belongs in next year's infra budget, one section for the engineers who'll actually wire it up: [link]

#Kubernetes #LLMInference #MLOps #PlatformEngineering #AIInfrastructure #CloudNative #vLLM #GenAI #DevOps #OpenSource
