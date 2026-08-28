# Chapter 20 — Hands-on: deploy your own service

Chapter 16 drew a line between an extension — no address, everyone gets
it automatically — and a service — has an address, someone calls it on
purpose. Everything so far in this Part has been the extension side of
that line: Istio, installed once, changing how every pod talks without
anyone asking it to. This chapter is the other side. `whoami` — a tiny
container from [`traefik/whoami`](https://hub.docker.com/r/traefik/whoami)
that does nothing but echo back whatever request it receives — is a real,
teaching-scoped service already deployed in this project, two different
ways at once, specifically so the two ways can sit side by side.

## Step 1: two sources, one identical service

```console
$ cat charts/whoami/Chart.yaml
apiVersion: v2
name: whoami
...
appVersion: "v1.10.3"

$ cat manifests/whoami-kustomize/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
```

`charts/whoami` is a real, minimal Helm chart — `Chart.yaml`,
`values.yaml`, and templates that use `{{ .Release.Name }}` and
`.Values`. `manifests/whoami-kustomize` is plain YAML — no templating, no
values file, just a `Deployment` and a `Service` listed directly.

## Step 2: how each is wired into Argo CD

```console
$ diff environments/platform-sandbox/whoami-helm.yaml environments/platform-sandbox/whoami-kustomize.yaml
```

The one field that differs in `spec.source`, beyond the path, is
`helm.releaseName`. `whoami-helm.yaml` sets it, because a Helm chart's
object names come from the release name and there's no other name for
Argo to use. `whoami-kustomize.yaml` doesn't need one — a plain Kustomize
target's object names are just whatever's written directly in the YAML.
Everything else — `repoURL`, `targetRevision: main`, `destination`,
`syncPolicy` — is identical. Both, notice, source from a path inside this
same repo, unlike Istio's Applications from Chapter 17, which point at an
entirely external Helm repository. This is the same pattern Chapter 10's
context map described — a thin, deliberate interface — applied to two
different shapes of "where the actual manifests live."

## Step 3: confirm both are running for real

```console
$ kubectl --context kind-pe-sandbox get applications whoami-helm whoami-kustomize -n argocd
NAME               SYNC STATUS   HEALTH STATUS
whoami-helm        Synced        Healthy
whoami-kustomize   Synced        Healthy

$ kubectl --context kind-pe-sandbox get pods,svc -n platform-services
NAME                                      READY   STATUS    RESTARTS   AGE
pod/whoami-helm-569fd9f768-4v87s          1/1     Running   0          21h
pod/whoami-kustomize-678bd6b79-t98bp      1/1     Running   0          21h

NAME                        TYPE        CLUSTER-IP      PORT(S)
service/whoami-helm         ClusterIP   10.104.1.189    80/TCP
service/whoami-kustomize    ClusterIP   10.109.253.80   80/TCP
```

This is the actual, live state of the sandbox cluster right now, not a
simulated example — both Applications synced and healthy, both pods
running.

## Step 4: talk to both, and see they're genuinely different pods

```console
$ kubectl --context kind-pe-sandbox -n platform-services port-forward svc/whoami-helm 18080:80 &
$ curl -s localhost:18080
Hostname: whoami-helm-569fd9f768-4v87s
IP: 127.0.0.1
IP: 10.244.1.19
...

$ kubectl --context kind-pe-sandbox -n platform-services port-forward svc/whoami-kustomize 18081:80 &
$ curl -s localhost:18081
Hostname: whoami-kustomize-678bd6b79-t98bp
IP: 127.0.0.1
IP: 10.244.1.20
...
```

Each response's `Hostname` line names the actual pod that answered —
proof these are two independently running services, not one service
reached two different ways. This is what "has an address, someone calls
it" from Chapter 16 looks like from the caller's side: a deliberate
request, to a known address, answered by one specific pod that exists
because of exactly one of these two Argo Applications.

## Step 5: close the loop Chapter 18 opened

Chapter 18 left something unresolved on purpose: every policy check in
this Part had technically passed only because the CI pipeline never saw
a real Deployment, just the Argo `Application` wrapper objects around
one. `charts/whoami` is the first thing in this repo you can render into
an actual `Deployment` locally, which makes it the first thing that can
prove `deny-latest-tag.rego` catches something real.

First, with the version this chart actually pins:

```console
$ helm template whoami-helm charts/whoami > /tmp/whoami.yaml
$ grep image: /tmp/whoami.yaml
          image: "traefik/whoami:v1.10.3"

$ conftest test -p policy /tmp/whoami.yaml
4 tests, 4 passed, 0 warnings, 0 failures, 0 exceptions
```

Now deliberately break it — bump the same rendered manifest's tag to
`:latest` — and run the identical check again:

```console
$ sed 's/traefik\/whoami:v1.10.3/traefik\/whoami:latest/' /tmp/whoami.yaml > /tmp/whoami-bad.yaml
$ grep image: /tmp/whoami-bad.yaml
          image: "traefik/whoami:latest"

$ conftest test -p policy /tmp/whoami-bad.yaml
FAIL - /tmp/whoami-bad.yaml - main - Deployment whoami-helm uses :latest tag in container whoami

4 tests, 3 passed, 0 warnings, 1 failure, 0 exceptions
```

Same rule. Same command. Every other time this book has run `conftest`
against something in this repo, the pass was real but empty — nothing to
check, so nothing failed. This time there's an actual `Deployment` in the
input, and the rule does exactly what Chapter 18 described it doing:
catches an unpinned tag, names the exact Deployment and container, and
fails the check. This is the first point in this whole book where a
policy rule has caught something real, not just something hypothetical.

<details>
<summary><strong>Predict before reading on:</strong> the manifest sourced from <code>manifests/whoami-kustomize</code> would pass this same check too, pinned or not — for a different reason than the Helm one passing did in Chapter 18. Why?</summary>

Because `manifests/whoami-kustomize/deployment.yaml` is plain YAML with no
templating at all — `kubectl kustomize` on it produces a real
`Deployment` object directly, the same as `helm template` does for the
Helm chart. Unlike the Istio `Application` wrapper objects from Chapter
18, there's no missing layer here: `conftest` run against this repo's
Kustomize output has always had a real Deployment to check, for both
`whoami-helm` and `whoami-kustomize`. The coverage gap was specific to
manifests sourced from an *external* Helm repository, like Istio's — not
to every check in this repo. Worth noticing precisely where a gap does
and doesn't apply, rather than assuming the whole pipeline is either
fully covered or not covered at all.
</details>

## Part 5 recap

> **Part 5 — The real workload**
> - ✅ Self-service — reinforced: deploying `whoami` two different ways
>   is still just editing a file and letting Argo CD do the rest, no
>   ticket, no manual `kubectl apply`
> - ✅ Golden paths — the base-plus-overlay shape from Chapter 16, and two
>   working reference patterns (Helm-sourced, Kustomize-sourced) this
>   chapter's own comments point a future service at
> - ✅ Developer experience — shown in this chapter: comparing both
>   sources side by side, then confirming both work, needed nothing more
>   than `kubectl` and `curl`
> - 🔲 Built-in operability — still the same real, current gap named back
>   in Chapter 2: the smoke test in Chapter 19 proves something worked
>   *right now*, not that anyone would notice if it broke at 2 a.m.
> - ✅ Progressive delivery — the forward pointer from Part 3 Chapter 11,
>   fully paid off in Chapter 17: the identical pattern, applied to an
>   Istio version bump moving through environments one at a time
> - ✅ Governance at scale / policy as code — the forward pointer from
>   Part 2 Chapter 6, fully paid off: named as a real, running check in
>   Chapter 18, then proven against an actual Deployment for the first
>   time in this chapter
> - ✅ Toil — named for the first time in Chapter 19, and shown: a script
>   that replaces watching a dashboard and hand-curling a URL with one
>   deterministic pass/fail
> - 🔲 The `auto`-image incident (Chapter 17) and the masked test-failure
>   incident (Chapter 19) — both told honestly, both queued for one
>   shared SRE retrospective in Part 6
> - 🔲 Trigger vs. root cause, blameless postmortems — still queued from
>   Part 3, and now with two more incidents for Part 6 to apply them to

Extensions and services, installed and checked the same disciplined way
everywhere else in this book has installed and checked things — a rule
written once, a script that replaces a person's repeated attention, and
two working examples you ran yourself rather than read about. Part 6
picks up from here with the two incidents this Part told honestly, and
asks what an SRE would actually do with them.

---

**Next: Part 6 — Closing the loop**
