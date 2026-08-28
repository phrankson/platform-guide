# Chapter 15 — Multi-tenancy: onboarding a second tenant

## One filing cabinet, more than one drawer

Chapter 14 traced the chain for exactly one tenant, `platform-services`.
The word "tenant" was doing real work there, borrowed from
**multi-tenancy**: the pattern of running several independent teams on
one shared piece of infrastructure, isolated enough from each other that
neither has to know the other exists. This chapter is about what actually
happens the moment a second one shows up.

## What onboarding actually requires

Say a team called `checkout` wants their own workloads running in
`platform-sandbox`, deployed the same GitOps way `platform-services`
already is. Following this repo's own onboarding recipe, on a working
copy of this repo:

```console
$ mkdir -p environments/platform-sandbox/tenants/checkout
```

A `kustomization.yaml` for that tenant, following the exact same shape
`platform-services`'s own folder already uses:

```console
$ cat environments/platform-sandbox/tenants/checkout/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - checkout.yaml
```

An `Application` object pointing at that team's own repo — this is
`checkout.yaml`, referenced above:

```console
$ cat environments/platform-sandbox/tenants/checkout/checkout.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: checkout
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/phrankson/checkout.git
    targetRevision: main
    path: environments/platform-sandbox
  destination:
    server: https://kubernetes.default.svc
    namespace: checkout
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

And exactly one line added to the tenant list from Chapter 14 — the only
shared file this whole process touches:

```console
$ git diff -- environments/platform-sandbox/tenants/kustomization.yaml
diff --git a/environments/platform-sandbox/tenants/kustomization.yaml b/environments/platform-sandbox/tenants/kustomization.yaml
index 6df1403..7f184ab 100644
--- a/environments/platform-sandbox/tenants/kustomization.yaml
+++ b/environments/platform-sandbox/tenants/kustomization.yaml
@@ -5,3 +5,4 @@ kind: Kustomization
 
 resources:
   - platform-services
+  - checkout
```

That's the entire onboarding process. Rendering the chain locally proves
both tenants show up, and proves `platform-services`'s own folder never
had to change at all:

```console
$ kubectl kustomize environments/platform-sandbox
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  labels:
    env: platform-sandbox
  name: checkout
  namespace: argocd
spec:
  destination:
    namespace: checkout
    server: https://kubernetes.default.svc
  project: default
  source:
    path: environments/platform-sandbox
    repoURL: https://github.com/phrankson/checkout.git
    targetRevision: main
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  labels:
    env: platform-sandbox
  name: platform-services
  namespace: argocd
spec:
  destination:
    namespace: platform-services
    server: https://kubernetes.default.svc
  project: default
  source:
    path: environments/platform-sandbox
    repoURL: https://github.com/phrankson/platform-services.git
    targetRevision: main
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    retry:
      backoff:
        duration: 5s
        maxDuration: 3m
      limit: 5
    syncOptions:
    - CreateNamespace=true
```

Both `Application` objects, both stamped `env: platform-sandbox` by the
same root file Chapters 13 and 14 already covered, from a single new
folder and a single new line. From here, a normal pull request against
`main` is all that's left — no branch per environment to merge, per
Chapter 13's filing-cabinet layout, and no manual `kubectl apply` once
it's merged. Argo CD finds the new `Application` object on its next
reconciliation pass, the same loop Chapter 12 already showed running
continuously, and starts managing `checkout` the same way it's already
managing `platform-services`.

<details>
<summary><strong>Predict before reading on:</strong> what would <code>kubectl kustomize environments/platform-sandbox</code> show if the <code>checkout/</code> folder existed with a valid <code>kustomization.yaml</code>, but the one-line edit to <code>tenants/kustomization.yaml</code> was skipped?</summary>

Exactly what it showed before `checkout` existed — one `Application`, `platform-services`, nothing else. No error, no warning, exit code `0`. Kustomize only walks folders it's told to look at; an unlisted folder isn't a broken reference, it's simply invisible. This is the single most common way onboarding goes wrong in a project shaped like this one — the fix works, gets committed, and nothing happens, with no error anywhere to point at why.
</details>

## Why `platform-services` doesn't get a shortcut

It would be easy to assume the platform team's own workloads deserve a
faster or more direct path than an outside team's — skip the tenant
folder, wire it in some other way, since it's "trusted." This repo
deliberately does the opposite. The comment sitting at the top of
`platform-services`'s own `kustomization.yaml` says so directly:

```console
$ head -3 environments/platform-sandbox/tenants/platform-services/kustomization.yaml
# platform-services tenant, platform-sandbox environment.
# Dogfooding: platform-owned services go through this exact same GitOps
# path as any other tenant's workloads -- no special-casing.
```

`platform-services` is a tenant folder, like any other, going through the
identical chain Chapter 14 traced. If the platform team wants proof this
pattern actually works before asking another team to trust it with real
workloads, running their own stuff through the exact same path — no
special case, no shortcut only the platform team gets to use — is the
only honest way to get that proof. A shortcut here wouldn't just be
unfair to the next tenant; it would mean the platform team never actually
tested the path everyone else is required to use.

## Self-service, made concrete

Go back to Chapter 1's definition: self-service means getting what you
need from a platform by declaring it, not by filing a ticket and waiting
for a person to act on your behalf. Onboarding `checkout` above never
involved asking the platform team to do anything. No ticket, no Slack
message, no person clicking a button on another team's behalf — one
folder, one line, one pull request, reviewed the same way any other
change to this repo is reviewed. That's the whole mechanism, and it's the
same shape Part 1 and Part 2 already showed for creating a repo and
attaching branch protection: declare what you need in a file, open a pull
request, and a program — not a person — makes it real.

## Part 4 recap

> **What Part 4 added**
> - ✅ Self-service — shown in this chapter: onboarding `checkout` required
>   one folder and one line, no ticket, no platform-team engineer acting
>   on anyone's behalf
> - 🔲 Golden paths, developer experience, platform as a product — still
>   just named, from Chapter 2
> - 🔲 Built-in operability — still the same real gap named in Chapter 2
>   and again in Chapter 10; nothing in this Part changes that
> - ✅ Environment = a folder — shown in Chapter 13, closing the thread
>   Part 1 Chapter 3 opened and Part 3 Chapter 7 partially paid off
> - ✅ Environment = a label — the third meaning, shown for the first time
>   in this Part, in Chapter 13 and again in Chapter 14's rendered output
> - ✅ Reconciliation loop, sync status vs. health status — shown live in
>   Chapter 12, against this project's real, currently-running cluster
> - ✅ App of Apps, `selfHeal` — shown field-by-field in Chapter 14
> - ✅ Multi-tenancy — shown in this chapter, onboarding a second tenant
>   without touching the first tenant's folder at all

The environment thread that's been open since Part 1 is fully closed now
— cluster, folder, and label, all three shown as real, not just named.
Multi-tenancy is the one this Part opens instead of closes: Part 5, not
yet written, is a new-ish addition arriving through this exact same
tenant path, and its
[final chapter](../part-5-the-real-workload/20-hands-on-deploy-your-own-service.md)
is where you'll onboard one yourself, hands-on, the same way `checkout`
just was here.

---

**Next: Part 5 — The real workload**
