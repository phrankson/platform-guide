# Chapter 17 — Installing the mesh

## Three deliveries, in a specific order

A shared intercom-and-security system for a whole house doesn't arrive in
one delivery. The wiring goes in first. Then the central control panel
everything else reports to. Then the front-door panel that actually lets
someone outside talk into the house at all. Install them out of order and
the later pieces have nothing to plug into yet.

[`environments/base/istio-helm.yaml`](https://github.com/phrankson/platform-services/blob/main/environments/base/istio-helm.yaml)
installs Istio, the service mesh used here, as exactly three Argo
`Application` objects, in exactly that order:

- **istio-base** — the wiring. Custom resource definitions and the
  cluster-level pieces nothing else can function without.
- **istiod** — the control panel. The mesh's control plane, the thing
  every other piece reports to and takes instructions from.
- **istio-ingress** — the front-door panel. The gateway that lets traffic
  from outside the mesh in through one controlled entrance.

The order is enforced with a sync-wave annotation on each one:

```yaml
metadata:
  name: istio-base
  annotations:
    argocd.argoproj.io/sync-wave: "0"
---
metadata:
  name: istiod
  annotations:
    argocd.argoproj.io/sync-wave: "1"
---
metadata:
  name: istio-ingress
  annotations:
    argocd.argoproj.io/sync-wave: "2"
```

Argo CD applies wave 0, waits for it to report healthy, then moves to
wave 1, then wave 2. Installing the front-door panel before the control
panel exists would leave it with nothing to report to.

Notice what's missing from that file: a version number for any of the
three. That's the same base-plus-overlay idea Chapter 16 just named,
applied here for real. `environments/platform-sandbox/istio.yaml` is the
file that actually pins a version, patched in per environment:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: istio-base
  namespace: argocd
spec:
  source:
    targetRevision: 1.30.3
```

The same patch, with the same version, repeats for `istiod` and
`istio-ingress` in that same file. Bumping the mesh's version means
editing this one file, watching `platform-sandbox` run on it for a while,
then copying the identical edit into `app-dev`'s `istio.yaml`, then
`app-prod`'s — never touching `base/`.

## The forward pointer from Chapter 11, paid off

Chapter 11 ended with a specific promise: the same progressive-delivery
pattern shown promoting `platform-core`'s three houses would show up
again, in a completely different repo, moving an Istio version through
environments one at a time instead of everywhere at once. This is that
moment. The shape is identical — build the change, prove it in the
cheapest environment first, only then copy it forward — the only thing
that's different is what's being promoted. There, it was an entire
cluster's infrastructure. Here, it's three lines changing a chart
version.

Right now, honestly, only `platform-sandbox` has a real `istio.yaml`
pin — `app-dev` and `app-prod` don't have one populated yet, so there's
nothing to promote *to* just yet. That's not the pattern failing; it's
the pattern waiting for its second and third data points, the same way a
paved road is still a paved road before a second car has driven down it.

## Running, but not actually wired in

<details>
<summary><strong>Predict before reading on:</strong> after the first real install, <code>istio-ingress</code>'s pod sat in <code>ImagePullBackOff</code> — stuck trying to pull an image literally named <code>auto</code>. That's not a typo in this project's config; <code>auto</code> is the chart's own placeholder value, meant to be swapped for a real image automatically. What has to happen for that swap to actually occur, and what happens if it doesn't?</summary>

Istio's gateway chart ships with `image: auto` on purpose. The idea is
that istiod's own control plane intercepts the request to create that pod
and rewrites the placeholder into a real image before the pod ever
starts — nobody has to know the exact image tag by hand.

That rewrite is done by a Kubernetes feature called an admission
webhook — code that gets a chance to inspect or modify an object the
moment it's created, before it's stored. This project's first real
install found a webhook rule that looked like the obvious explanation: it
only fires on namespaces carrying a specific label
(`istio.io/rev: default`), and this project's `istio-system` namespace
didn't have it yet. A fix went out adding that label via
`managedNamespaceMetadata` on each Application's sync policy.

Here's the honest part worth sitting with: after digging further, that
label turned out not to be the actual explanation. A different webhook
rule in the same chart matched regardless of the label, and the pod
eventually came up fine on a simple delete-and-recreate, with the label
never actually the thing that changed. The real cause looks like istiod
needing a short window after starting up before it can correctly perform
this exact rewrite — a timing issue, not a missing setting. The label fix
stayed in the codebase anyway, because it's harmless and a reasonable
practice on its own, but the first explanation for *why* the pod started
working was wrong.

This is worth knowing as a general shape of investigation, not just a
fact about Istio: the first plausible-sounding explanation for an
incident is not automatically the correct one, and a fix that happens to
make the symptom go away is not proof the reasoning behind it was right.
A good investigation keeps checking evidence against the story even after
something starts working again, and says so plainly when the evidence
stops supporting the original explanation.
</details>

Confirming the real, currently-running mesh — the label is still in
place, harmless, and the pods are healthy regardless of whether it was
ever the actual fix:

```console
$ kubectl --context kind-pe-sandbox get pods -n istio-system
NAME                             READY   STATUS    RESTARTS   AGE
istio-ingress-567bc595f7-jqz8f   1/1     Running   0          43h
istiod-7ff49bfb66-xg5hx          1/1     Running   0          43h

$ kubectl --context kind-pe-sandbox get namespace istio-system -o jsonpath='{.metadata.labels}'
{"istio.io/rev":"default","kubernetes.io/metadata.name":"istio-system"}

$ kubectl --context kind-pe-sandbox get applications -n argocd
NAME                SYNC STATUS   HEALTH STATUS
istio-base          OutOfSync     Healthy
istio-ingress       Synced        Progressing
istiod              OutOfSync     Healthy
```

`OutOfSync` on `istio-base` and `istiod` here isn't a symptom of the
incident above — it reflects upstream chart changes Argo CD has noticed
since the last sync, unrelated to the image rewrite. Both still report
`Healthy`, which is the distinction that matters: sync status describes
whether the live state matches Git right now; health status describes
whether what's running is actually working.

Part 6, not yet written, comes back to this exact incident — alongside
one more from later in this Part — through a dedicated SRE lens: writing
it up the way a real postmortem would, separating what triggered it from
what actually caused it.

---

**Next:** [Chapter 18 — Policy as code](18-policy-as-code.md)
