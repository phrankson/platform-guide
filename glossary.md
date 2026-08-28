# Glossary

One line per term, linking back to the chapter that taught it with a real
example. Grows as the book grows — later Parts add to this file rather
than starting a new one.

## Part 1 — Foundations

- **Platform team** — a team whose customers are other engineering teams,
  not end users. Taught in [Chapter 1](book/part-1-foundations/01-why-platforms-exist.md#platform-team-vs-stream-aligned-team).
- **Stream-aligned team** — a team that owns a slice of the actual
  business and serves real end users. Taught in [Chapter 1](book/part-1-foundations/01-why-platforms-exist.md#platform-team-vs-stream-aligned-team).
- **Self-service** — getting what you need from a platform by declaring
  it, not by filing a ticket and waiting. Taught (and demonstrated) in
  [Chapter 1](book/part-1-foundations/01-why-platforms-exist.md#try-it-yourself--watching-a-platform-team-create-a-repo).
- **Golden path / paved road** — the well-supported default way of doing
  something, easy to follow and hard to get wrong by accident. Named in
  [Chapter 2](book/part-1-foundations/02-the-five-pillars.md).
- **Developer experience** — reducing friction and mental overhead for
  the people using a platform. Named in [Chapter 2](book/part-1-foundations/02-the-five-pillars.md).
- **Platform as a product** — treating a platform's output as a real
  product with real customers. Named in [Chapter 2](book/part-1-foundations/02-the-five-pillars.md).
- **Built-in operability** — a platform being observable and operable
  from the start, not bolted on later. Named as a real, current gap in
  this project in [Chapter 2](book/part-1-foundations/02-the-five-pillars.md#one-pillar-this-project-doesnt-actually-have).
- **Domain** — the whole problem a system is solving. Taught in
  [Chapter 3](book/part-1-foundations/03-bounded-contexts.md#the-idea-from-scratch).
- **Bounded context** — a boundary around one part of a system where a
  word has exactly one meaning. Taught in [Chapter 3](book/part-1-foundations/03-bounded-contexts.md#the-idea-from-scratch),
  demonstrated with a real `ls` comparison in the same chapter's
  [hands-on section](book/part-1-foundations/03-bounded-contexts.md#try-it-yourself--seeing-the-boundary-not-just-reading-about-it).
- **Ubiquitous language** — the consistent vocabulary that holds inside
  one bounded context, but isn't promised to travel outside it. Taught in
  [Chapter 3](book/part-1-foundations/03-bounded-contexts.md#the-idea-from-scratch).
- **Context map** — a small, deliberately narrow interface that lets two
  bounded contexts work together without either understanding the
  other's internals. Named in [Chapter 3](book/part-1-foundations/03-bounded-contexts.md#the-thin-interface-between-two-bounded-contexts);
  seen as a real object in Part 3.
- **Declarative infrastructure** — describing what should exist in a file,
  and letting a program make reality match it, rather than performing the
  steps by hand. Demonstrated throughout [Chapter 1's hands-on section](book/part-1-foundations/01-why-platforms-exist.md#try-it-yourself--watching-a-platform-team-create-a-repo).

## Part 2 — Governance

- **Branch protection** — a server-side rule GitHub enforces on every push
  to a branch, regardless of whose machine it came from or what local
  shortcuts they took. Taught in [Chapter 4](book/part-2-governance/04-permits-and-blueprints.md#the-building-code--githubbranchprotection).
- **Signed commit** — a commit cryptographically sealed by a private key
  only the committer holds, provable without trusting anyone's git
  config. Taught in [Chapter 4](book/part-2-governance/04-permits-and-blueprints.md#the-building-code--githubbranchprotection).
- **Two-layer enforcement** — pairing a fast, skippable local check with a
  slower, unskippable server-side one, rather than relying on either
  alone. Taught in [Chapter 5](book/part-2-governance/05-two-inspectors.md#two-layers-on-purpose).
- **Governance at scale** — a platform enforcing a rule uniformly, by
  machine, once no single person can hold every rule in their head.
  Named and shown in [Chapter 6](book/part-2-governance/06-governance-at-scale.md#the-pattern-named).
- **Policy as code** — the same governance-at-scale idea applied to what
  actually gets deployed, not just how code gets merged. Named in
  [Chapter 6](book/part-2-governance/06-governance-at-scale.md#one-layer-further-down);
  seen as a real, running check in [Chapter 18](book/part-5-the-real-workload/18-policy-as-code.md),
  and proven against a real Deployment for the first time in
  [Chapter 20](book/part-5-the-real-workload/20-hands-on-deploy-your-own-service.md#step-5-close-the-loop-chapter-18-opened).

## Part 3 — Building the house

- **GitOps controller** — a program that watches a Git repository
  continuously and keeps a cluster's actual state matching whatever's
  declared there, on its own, forever. Taught in [Chapter 10](book/part-3-building-the-house/10-the-smart-home-hub.md#infrastructure-as-code-vs-configuration-as-code).
- **Infrastructure as code vs. configuration as code** — two different
  problems easy to blur together: IaC fits things that change rarely,
  where tear-down-and-rebuild is an acceptable recovery plan;
  configuration-as-code fits things that change constantly, where a fast,
  in-place update history matters more. Taught in [Chapter 10](book/part-3-building-the-house/10-the-smart-home-hub.md#infrastructure-as-code-vs-configuration-as-code).
- **Progressive delivery** — moving a change through environments of
  increasing consequence one at a time, with a real check between each
  move, instead of pushing everywhere at once. Taught in [Chapter 11](book/part-3-building-the-house/11-proving-it-works.md#promoting-through-three-houses-with-two-checkpoints);
  the identical pattern shown again, applied to an Istio version bump, in
  [Chapter 17](book/part-5-the-real-workload/17-installing-the-mesh.md).
- **Inner loop / outer loop** — the inner loop is the fast, local
  edit-run-check cycle on a developer's own machine; the outer loop is
  everything after a change is pushed — build, deploy, and verify against
  a real environment. Taught in [Chapter 11](book/part-3-building-the-house/11-proving-it-works.md#an-inspector-who-doesnt-take-the-crews-word-for-it).
- **Trigger vs. root cause** — the trigger is the immediate event that set
  an incident off; the root cause is the underlying condition that made
  the trigger capable of causing damage. Named in [Chapter 10](book/part-3-building-the-house/10-the-smart-home-hub.md#the-flagship-incident-a-crash-looping-helper-three-houses-one-shared-limit)
  against the `fs.inotify` incident; applied to all four of this book's
  real incidents side by side in [Chapter 21](book/part-6-closing-the-loop/21-sre-retrospectively.md#the-four-incidents-side-by-side).

## Part 4 — The filing cabinet

- **GitOps** — the source of truth for what should be running lives in
  Git, and a tool continuously makes the real cluster match it, forever.
  Taught in [Chapter 12](book/part-4-the-filing-cabinet/12-what-argocd-actually-does.md#the-problem-this-solves).
- **Reconciliation loop** — Argo CD's continuous read-compare-correct
  cycle: read desired state, read actual state, apply any difference,
  repeat forever. Taught in [Chapter 12](book/part-4-the-filing-cabinet/12-what-argocd-actually-does.md#a-loop-not-a-command).
- **Sync status vs. health status** — sync status asks whether the
  cluster matches Git; health status asks whether the object is actually
  working, independently of that. Taught in [Chapter 12](book/part-4-the-filing-cabinet/12-what-argocd-actually-does.md#two-different-questions-sync-status-and-health-status).
- **Environment (three meanings)** — a whole cluster in `platform-core`,
  a folder in `platform-gitops`, and a label on every rendered object;
  all three correct at once. Taught in full in [Chapter 13](book/part-4-the-filing-cabinet/13-one-cabinet-many-drawers.md#three-concrete-things-one-word),
  closing the thread opened in [Chapter 3](book/part-1-foundations/03-bounded-contexts.md#watching-this-happen-with-one-real-word).
- **App of Apps** — an Argo `Application` whose entire job is producing
  more `Application` objects, discovered automatically as they render.
  Taught in [Chapter 14](book/part-4-the-filing-cabinet/14-the-app-of-apps-pattern.md#file-four-the-one-that-actually-does-something).
- **`selfHeal`** — the reconciliation loop reverting a manual, out-of-Git
  change back to what's declared, on its own. Taught in [Chapter 14](book/part-4-the-filing-cabinet/14-the-app-of-apps-pattern.md#selfheal-concretely).
- **Tenant / multi-tenancy** — several independent teams (or the platform
  team's own workloads) running on one shared environment, isolated by
  folder, neither aware of the other. Taught in [Chapter 15](book/part-4-the-filing-cabinet/15-multi-tenancy.md#one-filing-cabinet-more-than-one-drawer).

## Part 5 — The real workload

- **Extension vs. service** — an extension changes how the cluster's own
  systems behave for every pod automatically, with no address anyone
  calls; a service runs continuously with an address a caller uses on
  purpose. Taught in [Chapter 16](book/part-5-the-real-workload/16-extensions-vs-services.md#two-different-kinds-of-vendor-work).
- **Sync-wave ordering** — an Argo CD annotation that forces a group of
  Applications to apply in a specific sequence, waiting for each wave to
  report healthy before starting the next. Taught in [Chapter 17](book/part-5-the-real-workload/17-installing-the-mesh.md#three-deliveries-in-a-specific-order).
- **Admission webhook** — code that gets a chance to inspect or modify a
  Kubernetes object the moment it's created, before it's stored; the
  mechanism istiod uses to rewrite the `auto` placeholder image into a
  real one. Taught in [Chapter 17](book/part-5-the-real-workload/17-installing-the-mesh.md#running-but-not-actually-wired-in).
- **Coverage gap** — a passing policy check can mean "checked, and it's
  fine" or "checked, and there was nothing here for the rule to apply
  to"; confusing the second for the first produces a false sense of
  governance coverage. Named honestly in [Chapter 18](book/part-5-the-real-workload/18-policy-as-code.md#a-rule-thats-only-checking-one-layer-of-the-truth),
  resolved for real in [Chapter 20](book/part-5-the-real-workload/20-hands-on-deploy-your-own-service.md#step-5-close-the-loop-chapter-18-opened).
- **Toil** — manual, repetitive work that doesn't get any easier no
  matter how many times you do it by hand; automating it saves more than
  just time on any one run. Named and shown in [Chapter 19](book/part-5-the-real-workload/19-toil-and-verification.md#forcing-the-check-instead-of-waiting-for-it).
- **Pre-merge / post-merge split** — validating a change before it can
  reach `main` with no real cluster involved, versus forcing
  reconciliation and smoke-testing against the real cluster only after
  merge. Taught in [Chapter 19](book/part-5-the-real-workload/19-toil-and-verification.md#verification-and-where-this-fits-in-the-pipeline).

## Part 6 — Closing the loop

- **Blameless postmortem** — reviewing an incident by asking what about
  the *system* allowed the failure, not who caused it, because the
  interesting finding is usually what let a reasonable action produce a
  bad outcome, not a person's mistake. Applied to all four of this book's
  real incidents in [Chapter 21](book/part-6-closing-the-loop/21-sre-retrospectively.md#four-incidents-one-pattern-underneath).
- **Error budget** — the amount of unreliability a team explicitly agrees
  is acceptable over some window, measured against a target, so that
  "ship this risky change" or "stop and fix reliability instead" has a
  number behind it. Named as a real gap this project doesn't have in
  [Chapter 21](book/part-6-closing-the-loop/21-sre-retrospectively.md#toil-and-error-budgets),
  closed out honestly in [Chapter 22](book/part-6-closing-the-loop/22-whats-still-missing.md#built-in-operability-one-last-time).
- **Platform ops** — a team or system responsible for a platform's own
  health the way an SRE team is responsible for a product's uptime:
  service-level objectives on the platform's own components, alerting
  that actually pages someone, an on-call rotation, and dashboards built
  for a glance. Named precisely, and named as missing from this project,
  in [Chapter 22](book/part-6-closing-the-loop/22-whats-still-missing.md#built-in-operability-one-last-time).
