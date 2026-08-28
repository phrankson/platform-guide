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
