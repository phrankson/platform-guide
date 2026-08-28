# Chapter 10 — The smart-home hub: installing Argo CD

## Installing a hub, then stepping back

Imagine the construction crew's very last job, once the house itself is
built: install a smart-home hub — one device that, once plugged in and
paired to an account, spends the rest of its life listening for
instructions from that account and carrying them out on its own. The
crew's job is to physically install the hub and pair it. It is
emphatically not the crew's job to personally rearrange the furniture
every time the homeowner wants something moved. That's what the hub is
for.

[`modules/argocd.py`](https://github.com/phrankson/platform-core/blob/main/modules/argocd.py)
is this repo's last act per house, and its own docstring says the
boundary out loud:

> Pulumi's job stops at getting the controller running. Once Argo CD is
> up, it takes over watching Git and reconciling application manifests on
> its own loop — Pulumi should never again touch a resource Argo CD owns.

## Infrastructure as code vs. configuration as code

This is worth naming precisely, because the two ideas are easy to blur
together: infrastructure as code and configuration as code solve
different problems, and mixing them up creates real pain. Pulumi is
excellent at "build the house and its foundation" — things that change
rarely, where "tear it down and rebuild" is an acceptable recovery plan,
the same target-replace move Chapter 9 just showed. It's a poor fit for
"what furniture is currently arranged where" — things that change
constantly, where you want a fast, cheap, in-place update history and an
easy rollback, not a foundation-level rebuild.

Argo CD — a **GitOps controller** — exists specifically for that second
job: it watches a Git repository continuously and keeps the cluster's
actual state matching whatever's declared there, forever, with no human
running a deploy command each time something changes.

Two functions in `argocd.py` do exactly the two things a crew does with a
smart-home hub:

**`install()`** physically installs the hub — a one-time Helm chart
install, run once by Pulumi, exactly like installing any other appliance:

```python
return helm.v3.Release(
    "argocd",
    helm.v3.ReleaseArgs(
        chart="argo-cd",
        version=version,
        repository_opts=helm.v3.RepositoryOptsArgs(
            repo="https://argoproj.github.io/argo-helm",
        ),
        namespace=namespace,
        create_namespace=True,
    ),
    opts=ResourceOptions(provider=provider),
)
```

**`seed_gitops()`** pairs the hub to an account — creating exactly one
object, an Argo `Application`, that tells the freshly-installed hub which
account to start listening to:

```python
argocd_root_app = argocd.seed_gitops(
    k8s,
    repo_url="https://github.com/phrankson/platform-gitops.git",
    path=f"environments/{pulumi.get_stack()}",
    depends_on=[argocd_release],
)
```

That object — nothing but a repository URL and a path — is the context
map Chapter 3 named and promised you'd meet for real: a deliberately thin
interface between two bounded contexts. `platform-core` never needs to
understand `platform-gitops`'s folder structure or tenant setup on the
other side of it, and `platform-gitops` never needs to know anything
about Docker networks or Kind. The instant this object exists, this
repo's job is finished for this house. Every deployment onward is Argo CD
discovering the next instruction in `platform-gitops` on its own — not
another `pulumi up`.

Notice `pulumi.get_stack()`, not the cluster's own name — a distinction
worth holding onto, and the naming wrinkle flagged at the end of Chapter
7. `platform-gitops`'s folders are named after the Pulumi stack
(`platform-sandbox`, `app-dev`, `app-prod`); the underlying Kind house
names differ slightly (`pe-sandbox` for the sandbox house). Using the
wrong one would pair the hub to a folder that doesn't exist.

## Argo CD instead of Flux

Flux is the more commonly taught GitOps controller, and it works
differently in a way worth understanding, since Argo CD is what this
project actually runs. Flux needs two separate objects talking to each
other: a `GitRepository` object that says which account to watch, and a
`Kustomization` object that says what to do with what that account says.
Argo CD's single `Application` object does the job of both, because it
bakes the account's address directly into the same object that says
where to deploy. There's no separate "register this account first" step
to manage.

## The flagship incident: a crash-looping helper, three houses, one shared limit

<details>
<summary><strong>Predict before reading on:</strong> the first real attempt to install this hub failed with the installer stuck, and a helper process called <code>redis-secret-init</code> crash-looping with the error <code>dial tcp 10.96.0.1:443: i/o timeout</code> — a total inability to reach the house's own internal address book from inside the house. Nothing about the hub's own settings was wrong. What single thing, shared across all three houses at once, was the actual cause?</summary>

The real culprit was `kube-proxy` — the component that makes a house's
internal address book (`10.96.0.1`) actually resolve to anything —
crash-looping on every node of all three houses simultaneously, with the
telling error `"command failed" err="failed complete: too many open
files"`.

The root cause was one shared limit on the machine itself, not anything
inside any single house:

```console
$ sysctl fs.inotify.max_user_instances
fs.inotify.max_user_instances = 128
```

Think of this as the shared electrical panel for the whole property, not
a per-house circuit — `128` is comfortably enough capacity for one house
under construction. Building all three simultaneously — each with its
own address-book service, its own network watcher, its own
name-resolution service, all needing to plug into the same shared panel —
tripped the breaker for the entire property at once. Once the internal
address book stopped resolving, nothing inside any of the three houses
could find anything else: not the hub's installer, not name resolution,
nothing. The visible symptom — the hub failing to install — sat several
layers downstream of the actual cause, a property-wide electrical limit
with zero connection to Argo CD, Helm, or this codebase.

The fix was a one-time, property-wide change, not a code change:

```console
$ sudo sysctl fs.inotify.max_user_watches=524288
$ sudo sysctl fs.inotify.max_user_instances=512
```

After that, `kube-proxy` recovered on its own on the next restart cycle,
and the exact same install that had just failed succeeded cleanly on
retry, with zero code changes. Checking this machine's own limits right
now confirms the fix is still in place:

```console
$ sysctl fs.inotify.max_user_instances fs.inotify.max_user_watches
fs.inotify.max_user_instances = 512
fs.inotify.max_user_watches = 524288
```

Worth keeping as a general instinct, not just a fact about this project:
the deepest layer of a stack — here, the host's own kernel limits — can
produce a symptom that looks exactly like a misconfiguration three layers
up, and the only way to tell the difference is checking the layer the
error message never mentions at all.
</details>

Part 6 of this book, not yet written, revisits this exact incident
through a different lens entirely: how Site Reliability Engineering would
actually write it up. A good postmortem stays blameless — it asks what
about the *system* allowed the failure, not who caused it — and it
separates the trigger from the root cause. The trigger here was running
three Kind clusters at once. The root cause was a host kernel limit
nobody had ever needed to think about before that point. Fixing only the
trigger (say, never running more than one cluster at a time) would have
avoided this specific incident without addressing the actual constraint.
Fixing the root cause, which is what actually happened, means the same
failure won't come back the next time something else pushes against that
same limit. Hold onto that distinction — Part 6 builds a whole chapter
around it.

## Seeing the whole chain, live, right now

```console
$ kubectl --context kind-pe-sandbox get applications -n argocd
NAME                SYNC STATUS   HEALTH STATUS
istio-base          OutOfSync     Healthy
istio-ingress       Synced        Progressing
istiod              OutOfSync     Healthy
platform-gitops     Synced        Healthy
platform-services   Synced        Healthy
whoami-helm         Synced        Healthy
whoami-kustomize    Synced        Healthy
```

`platform-gitops` is the one object `seed_gitops()` created directly —
the pairing instruction. Every other row in that list, this repo never
created; the hub found those on its own, by following `platform-gitops`.
Part 4 covers what those actually are and how the hub discovers them.

## The gap this Part is honest about

It's worth being direct about something the last few sections gloss over.
Once Argo CD is installed and paired, it keeps running on its own, but
nothing in this project watches it. If Argo CD's own pods crashed at 2
a.m., nothing would notice, and nothing would page anyone. Chapter 11's
tests check that things worked at the moment they were run, not
continuously. Platform ops — someone or something responsible for the
platform's own health, the way an SRE team is responsible for a product's
uptime — doesn't exist here yet. This Part ends at "prove it worked just
now," not "make sure it keeps working." That's the same built-in
operability gap Chapter 2 named as real and current; this is what it
looks like up close, not just as a line item on a list.

---

**Next:** [Chapter 11 — Proving it works](11-proving-it-works.md)
