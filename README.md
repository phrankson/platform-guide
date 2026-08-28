# platform-guide

A beginner-to-intermediate guide to this platform, written as one
continuous book instead of one document per repo. If you've read a
repo's `learning/` folder and then its `docs/` folder and found yourself
re-orienting every time you switched between them, this book is for you
— it tells the same story, but as one sequence, organized by topic
instead of by repo boundary, with concept, analogy, and hands-on practice
sitting next to each other instead of split across two documents.

## Who this is for

Someone who wants to actually understand platform engineering, not just
operate this specific project — a technical product manager, a new
platform engineer, anyone comfortable with a terminal but new to
Kubernetes, GitOps, or the frameworks platform teams use to talk about
their own work. No prior context on this project is assumed.

## How this differs from each repo's own docs

Every repo in this project also has its own `learning/` folder (a
narrative teaching companion, written first, one file per repo) and
`docs/` folder (task-oriented reference material for engineers actually
operating that repo). Neither is retired or replaced by this book. Think
of it this way:

- **This book** — read start to finish, once, to build the whole mental
  model.
- **Each repo's `learning/`** — a deeper, repo-scoped version of the same
  narrative, useful once you already have the big picture and want more
  depth on one specific repo.
- **Each repo's `docs/`** — reach for this once you're actually doing
  something in a repo and need the precise, current reference for a
  command, a schema, or a troubleshooting step.

## How to read a chapter

Three small devices repeat throughout the book:

- **`Predict before reading on` boxes** are collapsed on purpose — try to
  answer before opening them. Every command shown anywhere in this book
  was actually run against this project's real infrastructure; nothing
  is simulated.
- **Recap boxes** close every Part, checking off which concepts have
  actually been *shown*, not just *named*, so far.
- **The [glossary](glossary.md)** collects every term the book teaches,
  one line each, linking back to where it was taught with a real example.

## Table of contents

### Part 1 — Foundations ✅
1. [Why platforms exist](book/part-1-foundations/01-why-platforms-exist.md)
2. [The five pillars of platform engineering](book/part-1-foundations/02-the-five-pillars.md)
3. [Bounded contexts: why (eventually) six repos, not one](book/part-1-foundations/03-bounded-contexts.md)

### Part 2 — Governance (`platform-team-administration`) ✅
4. [Permits and blueprints — repos and branch protection as code](book/part-2-governance/04-permits-and-blueprints.md)
5. [Two inspectors — git hooks and CI/CD](book/part-2-governance/05-two-inspectors.md)
6. [Governance at scale — the same pattern, two layers](book/part-2-governance/06-governance-at-scale.md)

### Part 3 — Building the house (`platform-core`) ✅
7. [Three houses, not three rooms](book/part-3-building-the-house/07-three-houses-not-three-rooms.md)
8. [Utility hookups — networking, and the subnet that didn't match](book/part-3-building-the-house/08-utility-hookups.md)
9. [Pulumi's escape hatch, and the multi-cluster race](book/part-3-building-the-house/09-pulumis-escape-hatch.md)
10. [The smart-home hub — installing Argo CD, and the inotify incident](book/part-3-building-the-house/10-the-smart-home-hub.md)
11. [Proving it works — tests, progressive delivery, inner/outer loop](book/part-3-building-the-house/11-proving-it-works.md)

### Part 4 — The filing cabinet (`platform-gitops`)
12. What Argo CD actually does (the reconciliation loop)
13. One cabinet, many drawers — environments explained
14. The App of Apps pattern
15. Multi-tenancy — onboarding a second team

### Part 5 — The real workload (`platform-services`)
16. Extensions vs. services — what actually gets delivered
17. Installing the mesh, and the `auto`-image postmortem
18. Policy as code, and its honest coverage gap
19. Toil, verification, and the test that lied
20. Hands-on: deploy your own service (Helm vs. Kustomize)

### Part 6 — Closing the loop
21. SRE, retrospectively — every incident as one postmortem pattern
22. What's still missing, and why that's normal

Parts 4 through 6 are planned, not yet written — this book grows the same
incremental way every other part of this project does.

---

Start with [Chapter 1](book/part-1-foundations/01-why-platforms-exist.md).
