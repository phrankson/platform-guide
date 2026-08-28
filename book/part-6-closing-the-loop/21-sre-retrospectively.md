# Chapter 21 — SRE, retrospectively

## Four incidents, one pattern underneath

This book has been honest about four real incidents as they happened,
each in the chapter where it happened: a crash-looping helper that
blocked Argo CD's install ([Chapter 10](../part-3-building-the-house/10-the-smart-home-hub.md)),
an admission-webhook deadlock that never fully confirmed its own fix
([Chapter 4](../part-2-governance/04-permits-and-blueprints.md)), a
placeholder container image that wouldn't rewrite itself
([Chapter 17](../part-5-the-real-workload/17-installing-the-mesh.md)),
and a smoke test that reported success on a real failure
([Chapter 19](../part-5-the-real-workload/19-toil-and-verification.md)).
Each chapter told its own story and moved on, because that's what the
moment called for at the time. Site Reliability Engineering — the
discipline of running production systems, mostly built at Google and now
practiced widely — has a specific, repeatable way of going back over an
incident after the fact, and it's worth running all four through it
properly, side by side, instead of leaving each one as a standalone
story.

Two ideas do almost all the work.

**Blameless.** A postmortem asks what about the *system* allowed a
failure to happen — not who caused it. Not because blame is unkind, but
because it's usually wrong: a person following a normal process,
sysadmin instincts, or a plausible fix isn't the interesting finding.
The interesting finding is what let a reasonable action produce a bad
outcome, because that's the thing you can actually change.

**Trigger vs. root cause.** The trigger is the specific event that set an
incident off. The root cause is the underlying condition that made the
trigger capable of causing damage in the first place. Fixing only the
trigger stops this exact incident from happening again; fixing the root
cause stops the next one that would have used a different trigger to hit
the same weak spot. [Chapter 10](../part-3-building-the-house/10-the-smart-home-hub.md)
named this distinction first, against the `fs.inotify` incident, and
promised this chapter. Here it is, applied to all four.

## The four incidents, side by side

| Incident | Trigger | Root cause |
|---|---|---|
| `kube-proxy` crash-loop, Argo CD install stuck ([Ch. 10](../part-3-building-the-house/10-the-smart-home-hub.md)) | Three Kind clusters starting up at once | A host kernel limit (`fs.inotify.max_user_instances`) shared across all three, never sized for concurrent clusters |
| `enforce_admins` self-approval deadlock ([Ch. 4](../part-2-governance/04-permits-and-blueprints.md)) | The one GitHub account on the project tried to merge its own PR | `required_approving_review_count` hardcoded to `1`, quietly assuming a team size that didn't exist yet |
| `istio-ingress` stuck on the `auto` placeholder image ([Ch. 17](../part-5-the-real-workload/17-installing-the-mesh.md)) | The gateway pod got created before the admission webhook could rewrite its image | Not fully confirmed — see below |
| Smoke test reporting "Completed" on a real connection failure ([Ch. 19](../part-5-the-real-workload/19-toil-and-verification.md)) | `curl \| head` under `set -e`, checked at exactly the moment routing wasn't configured yet | Shell semantics: a pipeline's exit status is its *last* command's, and `head` exits 0 on empty input regardless of what `curl` did |

The kernel-limit incident and the deadlock both fit the blameless framing
cleanly, and both chapters already applied it without using the word.
Nobody misconfigured `fs.inotify` — the limit had simply never needed
raising before three clusters ran at once. Nobody wrote a bad governance
rule — `enforce_admins=True` and one required review are each correct in
isolation; they only collide with a team of one. In both cases, the fix
that stuck was the one that addressed the actual condition
(`sysctl`-raise the shared limit; make the review count a config value
instead of a constant), not the one that would have merely dodged the
next trigger (run one cluster at a time; turn off `enforce_admins`).

The smoke-test incident is the cleanest trigger/root-cause split of the
four, because the root cause is a fact about the shell, not about this
project's script specifically: any `pipe | head` under `set -e`,
anywhere, has this exact failure mode built in. The trigger was
incidental — it happened to surface the day routing wasn't configured
yet — but the root cause would have produced the same false "Completed"
on any other day a request failed for any other reason. That's precisely
why the fix removed the pipe rather than special-casing that one day's
failure.

## The one that stays honestly unresolved

The `auto`-image incident deserves more time than the other three,
because it's the one where the postmortem's honest ending is "we're not
fully certain" — and that's worth sitting with, not smoothing over.

<details>
<summary><strong>Predict before reading on:</strong> Chapter 17 described a namespace-label fix that shipped, and a pod that started working — but also said plainly that the label turned out not to be the actual explanation. If the fix that shipped wasn't the real fix, what does a blameless postmortem do with that?</summary>

It writes down exactly what Chapter 17 wrote down: the trigger is known
precisely (the gateway pod was created before the webhook could rewrite
its placeholder image), but the root cause is *not* fully confirmed. The
namespace-label theory was tested directly — added, and the pod still
needed a delete-and-recreate to actually come up — which is evidence
against it, not for it. The best remaining explanation is a short timing
window after istiod starts, before it's ready to correctly intercept and
rewrite pod creation. That's a reasonable, consistent-with-the-evidence
theory. It is not a confirmed root cause, and a real postmortem doesn't
upgrade it to one just because writing "confirmed" feels more finished
than writing "likely."

A blameless postmortem doesn't need a person to blame, and it doesn't
need a fully proven mechanism either, to still be useful. What it needs
is an honest account of what's known, what's suspected, and what's still
open — because the alternative is quietly retiring an incident's
uncertainty by writing a tidier ending than the evidence supports, which
is exactly the kind of thing that comes back to bite a team the next time
the same symptom shows up for a genuinely different reason.
</details>

This is the sharpest thing worth taking from all four incidents together:
a proper postmortem's job is to reach the correct confidence level for
what actually happened, not to reach a confident-sounding conclusion. The
kernel limit and the review-count deadlock both ended in fully confirmed
root causes — the evidence closed the loop cleanly. The `auto`-image
incident didn't, and the label fix stayed in the codebase anyway, because
it's harmless on its own merits even though it wasn't the load-bearing
fix. Writing that down honestly — "this is still running, this is
probably why, and here's what we tested that argues against a
competing theory" — is a complete, legitimate postmortem. It is not a
failure of the process to admit uncertainty; treating an untested guess
as a confirmed cause would have been the actual failure.

## Toil and error budgets

[Chapter 19](../part-5-the-real-workload/19-toil-and-verification.md)
named **toil** — manual, repetitive work that doesn't get easier no
matter how many times you do it — and showed a script replacing exactly
that kind of work with a deterministic pass/fail. SRE pairs toil with a
second idea worth naming here, even though this project doesn't use it
directly: an **error budget** is the amount of unreliability a team
explicitly agrees is acceptable over some window, tracked against a
measured target, so that "should we ship this risky change" or "should
we stop and fix reliability instead" has a number behind it instead of a
gut feeling.

This project doesn't have one. There's no service-level objective
written down anywhere for `whoami`, Istio, or Argo CD itself, and nothing
measuring uptime against one over time. That's worth naming plainly
rather than skipping past, because an error budget is what toil-reduction
is *for*, at a mature org: automating the toil (Chapter 19's contribution)
buys back engineering time; an error budget is what tells a team whether
the reliability that time bought is actually enough, or whether it needs
to keep going. This project has the first half. It doesn't have the
second half yet, and the next chapter is direct about what that means.

---

**Next:** [Chapter 22 — What's still missing, and why that's normal](22-whats-still-missing.md)
