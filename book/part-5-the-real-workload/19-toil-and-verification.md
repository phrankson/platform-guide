# Chapter 19 — Toil and verification

## Forcing the check instead of waiting for it

Argo CD checks this repo for changes on its own schedule, every few
minutes by default. That's fine most of the time, and genuinely annoying
during CI, where waiting several minutes just to find out whether a
change worked turns a two-minute pipeline into a ten-minute one for no
real reason.

[`scripts/argocd_reconcile.sh`](https://github.com/phrankson/platform-services/blob/main/scripts/argocd_reconcile.sh)
exists to skip that wait:

```bash
argocd app sync platform-services --core
argocd app sync istio-base istiod istio-ingress --core

argocd app wait platform-services istio-base istiod istio-ingress \
  --core --health --timeout 120
```

`argocd app wait` blocks until everything named is healthy or the timeout
is hit, and fails loudly if it isn't — a deterministic yes-or-no answer,
not something a person has to interpret by eye.

Site Reliability Engineering has a name for the kind of work this script
replaces: **toil** — manual, repetitive work that doesn't get any easier
no matter how many times you do it by hand. Watching a dashboard until a
sync finishes, then manually curling a URL to see if the result actually
works, is exactly that kind of task. It's worth naming precisely, because
the value of automating it isn't just the minutes saved on any one run.
Toil left alone doesn't shrink with practice the way a hard problem does
— it's the same tedious, error-prone attention every single time, and a
tired or distracted person doing it by hand eventually skips a step. This
is the same shape as the two-layer enforcement from Part 2: replacing "a
person remembers to check" with "a machine checks the same way every
time."

## A test that said yes when the real answer was no

The script's last job is a smoke test: push one real request through the
mesh's gateway and confirm it actually gets a response, not just that the
gateway's pod exists and is running.

<details>
<summary><strong>Predict before reading on:</strong> the very first version of this smoke test reported success — the job showed "Completed" — even though the gateway had no working route configured yet and every request to it was actually failing. How does a test end up reporting success on a real failure?</summary>

The test's script piped curl's output into `head`:

```bash
curl -sS -H "Host: $H" --max-time 10 http://... | head -n 3
```

The smoke test job runs under `set -e` — the shell exits immediately if
any command fails. But a pipeline's exit status under `set -e` is the
exit status of its *last* command, not any command earlier in the pipe.
`head` succeeds even when it receives no input at all — it just prints
nothing and exits cleanly. So no matter how badly `curl` failed —
connection refused, timeout, anything — `head`'s own success was the only
exit code `set -e` was ever watching. The job reported "Completed" while
quietly proving nothing at all.

The fix removes the pipe entirely, so there's nothing left to hide
behind:

```bash
curl -sS -H "Host: $H" --max-time 10 http://... -o /tmp/resp.txt
head -n 3 /tmp/resp.txt
```

Now curl's own exit code is the one `set -e` actually sees, and `head`
only ever runs on a response curl actually received. This is worth
remembering as a general habit, not just a fix for this one script: a
test is only as trustworthy as the exit code it actually reports on, and
a pipe can quietly swap which command that exit code belongs to.
</details>

**Try it yourself** — the corrected test, run against the real gateway,
correctly reporting a real failure (nothing has configured actual routing
through the mesh yet at this point in the book, so this failure is
expected, not a sign anything here is broken):

```console
$ kubectl --context kind-pe-sandbox -n istio-system apply -f smoke/http-gateway.yaml
$ kubectl --context kind-pe-sandbox -n istio-system wait --for=condition=complete job/smoke-gateway --timeout=30s
error: timed out waiting for the condition on jobs/smoke-gateway
$ kubectl --context kind-pe-sandbox logs -n istio-system job/smoke-gateway
curl: (7) Failed to connect to istio-ingress.istio-system.svc.cluster.local port 80 after 10 ms: Could not connect to server
```

A real failure, reported as a real failure — which is the entire point.
The corrected script's own comment says exactly why, right above the
line that matters:

```bash
# No pipe to head here on purpose: under `set -e`, a
# `curl | head` pipeline's exit status is head's, not
# curl's -- head reads nothing and still exits 0, so a
# real connection failure would silently report success.
```

## Verification, and where this fits in the pipeline

**Verification** is the general idea both of this chapter's tools serve:
checking a claim against reality, from the outside, rather than trusting
whatever made the claim. Chapter 11's `bats` tests were verification for
infrastructure — proof a cluster was really there, from a tool that
didn't build it. This smoke test is verification for a running
service — proof a request actually gets through, from a tool that didn't
deploy it.

[`.circleci/config.yml`](https://github.com/phrankson/platform-services/blob/main/.circleci/config.yml)
draws the same pre-merge/post-merge split covered in
`platform-team-administration`'s Part, adjusted for a repo with no
container image and no infrastructure of its own to provision:
**pre-merge** checks that every environment's configuration still builds
and passes policy, using no real cluster at all. **Post-merge** forces
the reconcile-and-smoke-test script above, against the real
`platform-sandbox` cluster, on the same self-hosted runner
`platform-core`'s pipeline uses — required because these Kind clusters
only exist on one physical machine, not in CircleCI's own infrastructure.

Only `platform-sandbox` is wired into the post-merge job today. `app-dev`
and `app-prod` don't have their own `istio.yaml` version pin yet, so
there's nothing real to reconcile there yet — the same honest gap Chapter
17 already named.

## Two incidents, one lens still to come

This chapter's masked-failure test and Chapter 17's `auto`-image
incident look unrelated on the surface — one's a shell scripting mistake,
the other's a Kubernetes admission-webhook timing issue. Part 6, not yet
written, comes back to both of them together, through a single SRE lens:
what actually failed, what the first plausible fix got wrong or right,
and what each incident says about the difference between a symptom
disappearing and a cause being understood. Hold onto both of them side by
side — that comparison is the point of that chapter, not something
either incident tells you on its own.

---

**Next:** [Chapter 20 — Hands-on: deploy your own service](20-hands-on-deploy-your-own-service.md)
