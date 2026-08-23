---
title: "The 8 Most Common AWS Cost Leaks — and What to Do About Each"
date: 2026-08-23
tags: [aws, cloud-cost-optimization, finops, devops, platform-engineering, engineering-leadership, cloud-economics]
permalink: /posts/common-aws-cost-savings-opportunities/
---

An AWS bill is a lot like a utility bill you never open the itemized version of. The total shows up every month, someone approves it, and life moves on. But when you finally sit down and read it line by line, the same pattern shows up in almost every company: a big chunk of the spend isn't paying for the product — it's paying for things nobody would knowingly keep if they saw them listed out. Old test environments. Storage that hasn't been touched in a year. Servers sized for a launch that happened in 2023. The good news is that the leaks are boring and repeatable, which means they're fixable without heroics.

Here are the eight places the money most commonly goes, in the rough order I see them show up.

## Where the money actually goes

**1. No commitment discounts on steady workloads.** AWS charges the highest price to customers who pay by the hour with no commitment — the "walk-in rate." The moment a workload is predictable (and most production workloads are), committing to it for one or three years through **Savings Plans** or **Reserved Instances** knocks 30–70% off the same compute, with no code change and no downtime. Companies routinely leave this on the table for months after they've already proven the workload is stable.

**2. Instances sized for what someone guessed a year ago.** Engineers pick an instance size when a service is first deployed, and it rarely gets revisited. Six or twelve months later, half the fleet is running at 5–15% CPU on hardware sized for 60%. Right-sizing — matching instance size to *actual* usage — is one of the fastest wins, because the data to do it has been sitting in AWS's own tooling the whole time.

**3. Servers running 24/7 for traffic that isn't.** This is the same problem the [KEDA post](/posts/keda-cost-savings-for-business-leaders/) covers on the Kubernetes side, and it's just as true for plain EC2 and containers on AWS. Dev and staging environments left on overnight. Batch jobs on machines that stay up between runs. Autoscaling that only scales *up*, never down. Matching capacity to real demand — including scaling to zero when nothing's happening — is often a bigger lever than any single discount.

**4. Storage that grew and never shrank.** S3 buckets fill up with data that was hot on day one and cold by day thirty, but stays in the expensive tier forever. EBS snapshots accumulate from backups nobody prunes. Log archives sit in standard storage when they belong in Glacier. Tiered storage — moving cold data to cheaper tiers automatically — often cuts storage spend by half or more without touching a single application.

**5. Zombie resources nobody remembers.** Every AWS account of a certain age has them: unattached Elastic IPs that bill by the hour, load balancers pointing at services that were decommissioned, NAT gateways sitting idle in unused VPCs, dev environments spun up for a demo in 2024. Individually small, collectively significant. A quarterly cleanup pass tends to find real money.

**6. Paying full price for workloads that don't need it.** Batch processing, CI runners, data pipelines, dev environments, machine-learning training — anything that can tolerate being interrupted and restarted — can run on **Spot instances** at 60–90% off. Companies often default everything to on-demand pricing simply because nobody's flagged which workloads are actually fault-tolerant.

**7. Data transfer costs nobody budgeted for.** AWS charges for data leaving its network, and — less obviously — for data moving between availability zones. A chatty microservice architecture that talks across AZs, a service that pulls large objects from S3 over the public internet instead of a VPC endpoint, or an egress-heavy API without a CDN in front of it can all quietly generate five- and six-figure line items. This is almost always fixable with routing changes, not code rewrites.

**8. Running on Intel when ARM is right there.** AWS's **Graviton** processors (ARM-based) offer roughly 20% better price-performance than the equivalent Intel instances, for a wide and growing set of workloads. Most modern managed runtimes — Java, Python, Node, Go, .NET — work on Graviton without changes. It's one of the highest-leverage migrations available right now: same performance, meaningfully lower bill.

## Why this matters beyond engineering

**It's a headcount conversation, not a spreadsheet one.** The dollar figures on a cloud bill can be abstract, but the equivalent in engineering salaries usually isn't. Trimming 20–30% off a mature AWS bill — which is a reasonable target for a company that has never done this work systematically — typically frees up enough budget for one or more full engineering roles. That's the frame that tends to make the work move.

**It's a hiring signal.** Engineers who think about cost tend to think about systems differently — they design for elasticity, they instrument what they build, they treat idle capacity as a bug. Roles that mention cloud economics or FinOps in the job description tend to attract that kind of candidate. And companies that visibly care about cost efficiency tend to hold onto them longer, because those engineers can see their work reflected in the business.

**It compounds.** Unlike a one-time contract renegotiation, most of these savings are structural. Once a workload is on a Savings Plan, right-sized, and running on Graviton, it keeps paying off every month without further attention. The discipline is the asset, not the individual fix.

## For the engineers reading this

If you're the person who'd actually implement this, here's the shortest-path checklist by category. None of these require third-party tooling to get started — everything below is native AWS.

- **Commitment discounts:** Start in **Cost Explorer → Savings Plans recommendations**. AWS analyzes your last 7/30/60 days and tells you the exact commitment amount with expected savings.
- **Right-sizing:** **AWS Compute Optimizer** and Cost Explorer's rightsizing recommendations. Both use CloudWatch metrics you're already collecting.
- **Autoscaling & scale-to-zero:** EC2 Auto Scaling groups with scale-in policies, scheduled scaling for predictable dev/staging patterns, [KEDA](/posts/keda-cost-savings-for-business-leaders/) on EKS for event-driven workloads.
- **Storage tiering:** **S3 Intelligent-Tiering** as the default for new buckets, **S3 Storage Lens** to find the worst offenders in existing ones, lifecycle policies to move logs to Glacier, EBS snapshot lifecycle manager to prune old snapshots automatically.
- **Idle resource cleanup:** **AWS Trusted Advisor** (cost optimization checks) for the low-hanging fruit; [`aws-nuke`](https://github.com/rebuy-de/aws-nuke) or [`cloud-nuke`](https://github.com/gruntwork-io/cloud-nuke) for programmatic teardown of ephemeral accounts. Tag everything with an owner and an expiration date on day one.
- **Spot instances:** EC2 Spot for stateless batch, **Karpenter** on EKS for mixed on-demand/spot fleets with graceful interruption handling, Spot for CI runners as an easy first win.
- **Data transfer:** **VPC endpoints** (Gateway endpoints for S3/DynamoDB are free) to keep traffic on the AWS backbone, **CloudFront** in front of egress-heavy APIs, and always check the AZ topology of chatty service-to-service traffic.
- **Graviton:** **AWS Graviton Ready** program lists compatible workloads; the [Graviton Getting Started Guide](https://github.com/aws/aws-graviton-getting-started) covers language runtimes and common gotchas. Managed services (RDS, ElastiCache, Lambda, Fargate) support Graviton with a single config change.

## The short version

The most common AWS cost leaks aren't exotic — they're the same handful of patterns showing up in almost every account. None of them require a re-architecture. Most of them can be closed with tooling AWS already ships. The reason they persist is almost never technical; it's that nobody has been given the time or the mandate to look.

If your team is trying to figure out where to start, or you're hiring for a role where this kind of thinking matters, I'm happy to talk through it in plain terms. Reach out any time.

Sources:
- [AWS Savings Plans](https://aws.amazon.com/savingsplans/)
- [AWS Compute Optimizer](https://aws.amazon.com/compute-optimizer/)
- [AWS Graviton Processors](https://aws.amazon.com/ec2/graviton/)
- [AWS Trusted Advisor cost optimization checks](https://aws.amazon.com/premiumsupport/technology/trusted-advisor/)
