# Chapter 22 — What's still missing, and why that's normal

## Built-in operability, one last time

[Chapter 2](../part-1-foundations/02-the-five-pillars.md) named built-in
operability as a real, current gap in this project, not a placeholder
waiting for a later chapter to quietly resolve it.
[Chapter 10](../part-3-building-the-house/10-the-smart-home-hub.md) showed
what that gap looks like up close, the moment Argo CD itself was
installed: nothing watches the watcher. Every Part 5 recap box since has
carried the same line forward, unchanged, next to four other pillars that
each eventually earned a ✅. This one never did. That's not an oversight
in how the recap boxes were kept — it's the actual state of the project,
checked again just now:

```console
$ kubectl --context kind-pe-sandbox get pods -n argocd
NAME                                                        READY   STATUS    RESTARTS   AGE
argocd-05961c3a-application-controller-0                    1/1     Running   0          43h
argocd-05961c3a-applicationset-controller-cdd4585d9-p8n4g   1/1     Running   0          43h
argocd-05961c3a-dex-server-9cccdffd8-bk4x6                  1/1     Running   0          43h
argocd-05961c3a-notifications-controller-7dcb949799-nrtpc   1/1     Running   0          43h
argocd-05961c3a-redis-5f8fbb9c96-ctbbd                      1/1     Running   0          43h
argocd-05961c3a-repo-server-5475b4fd4c-wnt6r                1/1     Running   0          43h
argocd-05961c3a-server-b9f57c4b4-m9gl4                      1/1     Running   0          43h
```

Every pod is healthy right now, at the moment this command happened to
run. That's exactly the shape of the gap: this output is a snapshot, not
a guarantee. Nothing in this project would produce this same table on its
own if `argocd-05961c3a-application-controller-0` crashed at 2 a.m. — no
alert would fire, no dashboard would turn red, no one would be paged.
Someone would find out the next time they happened to run this same
command, or the next time a developer noticed their deploy had silently
stopped working.

It's worth naming precisely what real **platform ops** looks like at an
organization that has this pillar, because "monitoring" undersells it.
A mature setup has service-level objectives for the platform's own
components (is Argo CD's reconciliation loop keeping up, not just is the
pod running), alerting wired to something that actually pages a human,
an on-call rotation so a page has somewhere to land at 2 a.m., and
dashboards built for a glance, not a `kubectl` command someone has to
remember to type. [Chapter 21](21-sre-retrospectively.md)'s error budget
is the measurement underneath all of that — the number that tells a team
whether platform ops is keeping up with what was promised, or whether
it's time to stop shipping features and fix reliability instead. None of
that exists here. This project can tell you, right now, by hand, whether
things are healthy. It cannot tell you on its own, without being asked.

That gap doesn't get smaller by writing this chapter. It gets named
clearly enough that finishing this book was never the same thing as
finishing the platform.

## Two bounded contexts that still don't exist

[Chapter 3](../part-1-foundations/03-bounded-contexts.md) listed six
repos this project's own configuration declares, called out
`platform-extensions` and `platform-demo-apps` as real but empty, and
called that a roadmap rather than a gap. Worth checking honestly, at the
end of the book, whether that's still a fair thing to call it:

```console
$ gh api repos/phrankson/platform-extensions --jq '{name, size, pushed_at}'
{"name":"platform-extensions","size":0,"pushed_at":"2026-08-21T16:02:09Z"}

$ gh api repos/phrankson/platform-demo-apps --jq '{name, size, pushed_at}'
{"name":"platform-demo-apps","size":0,"pushed_at":"2026-08-21T16:01:59Z"}
```

Both still `size: 0`. Both still exactly as empty as they were back in
Part 1. This book is now finished, and those two repos are not — which
means this book is only as complete as the project it documents, and the
honest answer is that the project has four working bounded contexts out
of six planned, not six. Nothing about that is a broken promise; Chapter
3 never said otherwise. But it's worth resisting the urge to let "planned
for later" quietly become "basically done" just because this is the
book's last page. It isn't done. Two contexts are still just an entry in
a YAML file and an empty repository, and a reader picking this project up
today would find exactly that if they went looking.

## The closing recap

Every Part before this one ended with a recap box checking off what had
actually been *shown*, not just named, so far. This is the last one,
covering the whole book instead of one Part.

> **The five pillars — final status**
> - ✅ **Self-service** — shown from [Chapter 1](../part-1-foundations/01-why-platforms-exist.md)
>   onward: every repo, permit, deploy, and service in this book got
>   created by declaring it in a file and opening a PR, never by filing a
>   ticket
> - ✅ **Golden paths** — shown fully in [Chapter 20](../part-5-the-real-workload/20-hands-on-deploy-your-own-service.md):
>   two working, ready-to-copy reference patterns (Helm-sourced,
>   Kustomize-sourced) for deploying a new service
> - ✅ **Developer experience** — shown in [Chapter 20](../part-5-the-real-workload/20-hands-on-deploy-your-own-service.md):
>   comparing and confirming two live services needed nothing more than
>   `kubectl` and `curl`
> - ✅ **Platform as a product** — shown in [Chapter 4](../part-2-governance/04-permits-and-blueprints.md):
>   the self-approval deadlock got fixed by making the rule honestly fit
>   the team that exists, not by weakening the guarantee
> - 🔲 **Built-in operability** — named in [Chapter 2](../part-1-foundations/02-the-five-pillars.md),
>   confirmed still missing in this chapter. The one pillar this book
>   ends without being able to check off
>
> **The concept spine — where each idea was taught**
> - **Bounded contexts** — [Chapter 3](../part-1-foundations/03-bounded-contexts.md),
>   with a real `ls` comparing repos side by side
> - **Context maps** — named in [Chapter 3](../part-1-foundations/03-bounded-contexts.md),
>   seen as one real object (a repo URL and a path) in [Chapter 10](../part-3-building-the-house/10-the-smart-home-hub.md)
> - **Governance at scale** — [Chapter 6](../part-2-governance/06-governance-at-scale.md),
>   the same pattern applied a layer further down as policy-as-code,
>   proven for real in [Chapter 20](../part-5-the-real-workload/20-hands-on-deploy-your-own-service.md)
> - **Environment, three meanings at once** — opened in [Chapter 3](../part-1-foundations/03-bounded-contexts.md),
>   closed in full in [Chapter 13](../part-4-the-filing-cabinet/13-one-cabinet-many-drawers.md)
> - **Progressive delivery** — named in [Chapter 11](../part-3-building-the-house/11-proving-it-works.md),
>   paid off with an Istio version bump in [Chapter 17](../part-5-the-real-workload/17-installing-the-mesh.md)
> - **Toil** — named and automated away in [Chapter 19](../part-5-the-real-workload/19-toil-and-verification.md)
> - **Trigger vs. root cause** — named against the `fs.inotify` incident
>   in [Chapter 10](../part-3-building-the-house/10-the-smart-home-hub.md),
>   applied to all four of this book's real incidents side by side in
>   [Chapter 21](21-sre-retrospectively.md)
> - **Blameless postmortems and error budgets** — this Part's own
>   additions, [Chapter 21](21-sre-retrospectively.md), closing the loop
>   this chapter opened on toil

Four pillars demonstrated. One left honestly open. Six bounded contexts
planned, four built, two still just a line in a config file. That's the
real, current state of this project — not a lesser ending than a tidy
one, just a truthful one, which is the same standard every incident in
this book got held to along the way.

---

**This is the end of the book.** If you want more depth on any one repo
than this book's pace allowed, each repo's own `learning/` folder is
still there, and the [glossary](../../glossary.md) still links every term
back to where it earned its example.
