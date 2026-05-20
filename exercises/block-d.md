# Block D — CI/CD pipelines that work; deployment strategy primer

**Duration:** ~75 minutes
* 25 min demo
* 15 min we-do
* 20 min you-do
* 15 min debrief

> This is a *discussion + demo* block. No code is committed; no checkpoint tag. The output is a worksheet you'll continue into the Day 2 → Day 3 homework.

## Goal

You should leave this block able to:

- Describe **rolling**, **blue/green**, and **canary** deploys, including their trade-offs
- Reason about which strategy fits a given context (SaaS, factory floor, batch job, Ignition gateway)
- Sketch what each strategy looks like *for your own* product — the start of your homework

## I do (25 min)

The instructor walks the three strategies side-by-side with diagrams. The same release scenario applied to each:

> *You're shipping version 1.5 of your product. Version 1.4 is in production. The migration is small but irreversible. How do you ship?*

### Rolling

Instances are replaced *gradually*. At any given moment, some traffic is on v1.4 and some on v1.5. New instances come online; old ones drain and shut down.

- **Pros:** No extra capacity needed. Smooth transition. Default in Kubernetes, ECS, etc.
- **Cons:** Hard to roll back partway through. Both versions must coexist (schema, contracts).
- **Fits:** Backend services with stateless requests and forward-compatible changes.

### Blue/green

Two complete environments — *blue* (current) and *green* (new). Deploy to green, smoke-test, then flip traffic from blue to green in one step. Blue stays around for instant rollback.

- **Pros:** Instant rollback. No version coexistence at the application layer. Conceptually simple.
- **Cons:** 2× capacity during deploy. Schema migrations are still tricky (the DB is shared).
- **Fits:** Critical services where rollback speed matters more than capacity cost. Many Ignition deployments fit this — gateways are often pets, and "spin up a second one" is feasible.

### Canary

Deploy to a small fraction of traffic (1%, then 5%, then 25%, then 100%). Watch metrics. Roll back at any percentage if anything degrades.

- **Pros:** Catches regressions with real users at limited blast radius. Best for risk-bounded shipping.
- **Cons:** Requires traffic-splitting machinery (service mesh, load balancer rules, feature flags). Slow. Hard for Ignition (no easy way to send "1% of HMI users" to v1.5).
- **Fits:** High-traffic services with mature observability. Not most Ignition shops.

The instructor demos one of the three live in a sandbox (instructor's choice — usually blue/green, since it's the most concrete to show).

## We do (15 min)

Three short scenarios on the board. For each, the class votes which strategy fits — and the instructor presses on edge cases.

1. **A factory-floor Ignition gateway** running production lines 24/7. New project version due Friday.
2. **A SaaS web product** with 100k DAU, ten engineers, mature observability.
3. **A nightly batch job** that takes 6 hours and processes the day's transactions.

Spoiler: the answers aren't all the same, and "it depends" is a legitimate answer with the right *why*.

## You do (20 min)

Fill out [`worksheets/deployment-strategy-worksheet.md`](../worksheets/deployment-strategy-worksheet.md) for **your own team's product**. Twenty minutes is enough for a rough draft; you'll refine it for the Day 2 → Day 3 homework.

If you don't have a current production product, sketch it for an Ignition client you've consulted with, or use the factory-floor scenario from "We do."

## Debrief (15 min)

Pair up. Swap worksheets. Read your partner's quietly first; then:

- What did your partner spot that you missed?
- Where did your partner's chosen strategy surprise you?
- Is there a scenario where the *right* answer is "we don't deploy continuously — and that's fine"?

## Homework hand-off

The Day 2 → Day 3 homework is to refine your worksheet into a one-page document and submit it as a PR to the cohort playground repo (per [`../cicd-masterclass/CURRICULUM.md`](../cicd-masterclass/CURRICULUM.md)). You've done most of the thinking already — tonight is editing and PR opening.
