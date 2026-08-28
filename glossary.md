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
  seen as a real, running check in Part 5.

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
  move, instead of pushing everywhere at once. Taught in [Chapter 11](book/part-3-building-the-house/11-proving-it-works.md#promoting-through-three-houses-with-two-checkpoints).
- **Inner loop / outer loop** — the inner loop is the fast, local
  edit-run-check cycle on a developer's own machine; the outer loop is
  everything after a change is pushed — build, deploy, and verify against
  a real environment. Taught in [Chapter 11](book/part-3-building-the-house/11-proving-it-works.md#an-inspector-who-doesnt-take-the-crews-word-for-it).
- **Trigger vs. root cause** — the trigger is the immediate event that set
  an incident off; the root cause is the underlying condition that made
  the trigger capable of causing damage. Named in [Chapter 10](book/part-3-building-the-house/10-the-smart-home-hub.md#the-flagship-incident-a-crash-looping-helper-three-houses-one-shared-limit)
  against the `fs.inotify` incident; Part 6 covers this fully.
