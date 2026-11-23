---
name: architecture-foundations
description: "Use BEFORE starting a new project, designing a new feature or subsystem, or reviewing a design, in any project: architecture soundness review, structuring a new system, avoiding a later rewrite, a design that keeps needing patches, one slow job blocking unrelated requests, fixes that each add another branch, a second model reviewing a design. Covers the method (requirement in the requester's words, real operating profile, inventory of lifetimes and owners, the invariant that removes a whole class of cases, one home per fact, exchanges that end explicitly, rollout that reaches running code, a comparative decision case, checks on the real system before building), the sources to consult and in what order, and how two independent models converge on one design. Routes state shapes to stateful-shapes, exchanges between parties to protocol-design, duplicated values to architecture-lessons, signals to application-observability, and review depth to review-gates. Do NOT use for picking a UI design or for a one-line bug fix."
---

# Architecture foundations

> What makes a bridge stand is not the bricks, it is the architecture. The people you build for
> will rarely ask for architecture by name; they ask that it not need to be rebuilt from scratch
> later. This skill is how to make that true.

This skill is the method for designing or reviewing the structure of a system or a feature so it
does not need a rewrite. It is generic: any project. It sits above the narrower skills and routes
to them; it repeats none of their content.

## When it applies

- Starting a project, a service, an app surface, a background job, an integration.
- Adding a feature that introduces a lifetime (a process, a session, a worker), a store, a
  channel between parties, a rollout path, or a value more than one place needs.
- Reviewing a design someone else wrote, including another model's.
- A fix that would add the third branch to the same code, or a design that gets more complex each
  iteration: stop and redesign from the invariant.

Not for: visual or interaction design; a bounded bug fix with an obvious path; a one-file script.

## The method, in order

1. **Write the requirement in the requester's words, then the operating profile.** Quote the request
   verbatim; list the written priorities that bind it (the project's decision notes and its
   agent instructions). State the real profile: users, machines, load, what sleeps, consequence of loss,
   reversibility. Never silently assume a fleet or many operators. A rule that trades the user's
   latency for the system's convenience is checked against those priorities before it is
   designed, not after it ships. (`references/invariants.md` §1, §2)
2. **Inventory what exists before inventing.** Every lifetime (which process holds what, who
   kills it, what dies with it), every owner of durable state, every channel between parties and
   what it already retries, every rollout path and whether it can reach code that is already
   running. Read the code and the supervisor config, not the docs' memory of them. Measure the
   numbers that decide the design (restart time, cost per call, message size) on the real system.
   For "the server is slow" work, attribute processor time per workload from retained per-minute
   evidence over the incident window before proposing any move, and ask whoever built an existing
   separation before building on or beside it. A short live sample is not evidence.
   For voice or other multimodal work, identify the recognizer, transcript transport, agent
   provider, continuing session and working project separately. A silent text transport does not
   establish that the requested agent or workspace is being used; check with a fresh agent in an
   isolated workspace before presenting it as the user's conversation.
3. **Find the invariant, not the branches.** Name the entity, the failure mode and the shape
   ([stateful-shapes](../stateful-shapes/SKILL.md)); name the exchange and its terminal states ([protocol-design](../protocol-design/SKILL.md)). A design is
   right when one invariant removes a whole class of cases ("execution lifetime is independent of
   coordinator lifetime" removes "wait for idle", "hold new work", "cancel on shutdown" at once).
   If each review round adds a branch, the invariant is wrong.
4. **Place every fact, lifetime and exchange.** One home per fact ([architecture-lessons](../architecture-lessons/SKILL.md)); one
   owner per lifetime with a stated stop condition and recovery event; every exchange ends in an
   explicit typed terminal state, never prose or silence; rollout can reach what is already running;
   the user's inputs survive any restart by identity. Fill in [the template](references/method.md).
5. **Make the comparative decision case.** Objective and acceptance criteria; the smallest
   candidate; the upstream or native path; doing nothing; where each wins and loses; the measured
   evidence that separates them; unknowns as unknowns with the smallest probe that settles each.
   Never call an option best without this. A second copy, a fallback list, a broker "just in case"
   or a phased activation is a candidate to argue against, not a default. Judge the candidates
   against what the requester has already said they want (the project's decision notes: for
   example event-triggered work or the simplest mechanism) and write the winning reason and what
   it beat into the design, the commit and the answer, so "why was this chosen?" has an answer
   later.
   For a bounded semantic judgment over options supplied by code (routing, ranking,
   classification, verification), consider a typed-output model call alongside rules, the current
   owner, and a native constrained model; a fast typed answer still needs outcome testing.
6. **Check before building, on the real system.** For each part: a probe that proves the
   assumption (a provider's behaviour, a supervisor's kill semantics, a client's retry), run
   harmlessly without interrupting real work; an acceptance check the user can see; the signal
   that tells an operator which state each exchange is in ([application-observability](../application-observability/SKILL.md)). How a
   third-party tool stores, caches and renews a credential is such an assumption: cite its source,
   then watch the tool against a stub that records every call it makes, before presenting the design
   to the user as the fix. Then
   decide the review level ([review-gates](../review-gates/SKILL.md)). Deliver each step whole, with its own acceptance; a
   dependency-ordered sequence of complete deliveries is not a phased activation of one feature.

For a page, change feed, cache or replay path, apply [bounded work](references/bounded-work.md):
name the growth variable, all writers, the serving process and its enforced read contract;
exercise the complete consumer under growth and restart. An indexed first lookup, a small
returned array or a component fixture alone does not prove the operation is bounded.

For "one slow job freezes every request", map execution as well as ownership. Calling a function
`async`, opening a separate database connection, or labelling a job background does not move its
synchronous work off the accepting thread. Prove independence by holding one executor in a
private composed fixture while unrelated authenticated reads and durable command admission
continue. Keep one mutation authority; isolate read execution without cloning schedulers or
claiming custody means a command committed. Name queue, cancellation, lifetime and saturation
behavior, and enforce the accepting import boundary in the build. The requirement is that one
request coming in is never stalled under another; a slow span that does not identify the blocking
call still shows the defect is the shared execution path. Logging cost and follow-up evidence belong to
[application-observability](../application-observability/SKILL.md), not another monitor here.

## Sources, in the order they pay off

1. The project's own retrospectives, incident notes, design plans and decision notes: the mistakes
   already paid for.
2. The code and the supervisor configuration (systemd units, launchd plists, the spawn call): what
   is actually true today.
3. Reading worth knowing for the pattern names in play (Kleppmann, the Anthropic harness split,
   Meta's control-plane split, Alan Kay): see [sources](references/sources.md).
4. The platform's official documentation and pinned source for every provider or OS primitive the
   design leans on ([sources](references/sources.md)). Third-party write-ups are hypotheses until the
   canonical page or pinned source agrees.
5. Established models only where they fit at this scale: AWS Well-Architected reliability and
   operational-excellence practices as principles (graceful degradation, loose coupling, idempotent
   mutations, bulkheads, automated rollback, bounded retries), Erlang supervision, Temporal signals,
   FIPA request, transactional outbox. Cite the practice id and say which mechanics do not apply
   to a small operation.

## Two independent models converging

Use [converge-two-models](references/converge-two-models.md). In short: the second model reviews the first's artifact
with the original words, verifies every "Confirmed" citation against current code, runs the probes
the first could not, and challenges with evidence; both record agreement and disagreement in one
document under both names; a disagreement that evidence cannot settle stays in the document
verbatim with each side's evidence, never papered over. Three rounds, then the human decides.

## Red flags that mean "stop and redesign"

- "We'll hold the user's input until X finishes." (A rule trading the user's latency for convenience.)
- "The updater will pick it up next cycle." (A rollout that cannot reach running code.)
- A classifier of free text standing in for an enforcement boundary.
- A fallback copy of a value that has an authoritative home.
- A phase plan where phase 1 ships half a feature.
- A "Confirmed" claim with no `path:line`, no URL, or a URL nobody fetched.
- A design that grew a subsystem the request never named.

## References

- `references/invariants.md`: the twelve invariants, each with the incident that taught it.
- `references/method.md`: the design document template (fill every section; unknowns need a probe).
- `references/converge-two-models.md`: the review-and-converge protocol between two models.
- `references/sources.md`: the canonical sources, with what each is good for.
- `references/bounded-work.md`: keeping the work a person waits for bounded.
