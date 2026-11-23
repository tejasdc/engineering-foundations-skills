# The invariants, with the incident behind each

Twelve rules. Each one exists because a real system broke without it. A short anonymous
illustration stays next to the rule where it helps; the rules are generic.

## 1. The requester's written priorities bind every design; a rule that trades the user's latency for the system's convenience is checked against them first

Example: to let updates install, a change held every new input, including the user's own
messages, until the longest-running job finished, and had to be reverted.

Rule: before a design makes anything wait, name whose latency it spends and check the project's
decision notes. Convenience for the system is never a reason for the user to wait.

## 2. State the real operating profile; never assume a fleet

Example: a single-operator webhook became a platform design with mandatory access control and
disaster recovery, and its kernel layer was reverted soon after.

Rule: write down users, machines, load, what sleeps, consequence of loss, reversibility. Size
every mechanism to that. Bulkhead cells and blue/green fleets are principles here, not mechanics.

## 3. Separate the lifetime of what does the work from the lifetime of what coordinates it

Example: every running job was a child process of the service, so the service could not restart
without killing them, and updates waited for an idle moment that never came. Anthropic's
managed-agents team hit the same wall and split harness from sandbox ("the harness leaves the
container"); Meta's XFaaS tolerates "controller downtime for tens of minutes" because the
execution path does not depend on the controller.

Rule: the thing you update often must not hold the lifetime of the thing that must keep running.
Name, per process, who holds its pipes, which supervisor kills it and what dies with it.

## 4. A rollout must be able to reach code that is already running

Example: a revert landed in the repository, but a self-updating client that was already running
kept executing the old policy until its next cycle; separately, a rollback to a binary that could
not open the new schema crash-looped.

Rule: for every long-running or self-updating part, state how a change reaches an instance that
is already executing (a stable entry that selects a versioned runner per attempt; a schema that
the previous release can still open). "Next cycle" is a design decision, not a default.

## 5. One home per fact, and something refuses the second copy

Example: a model list written in several places kept offering a retired model after an upgrade,
and a "derived" type guard had silently widened to `string`, so a deliberately broken build
passed. Full treatment: [architecture-lessons](../../architecture-lessons/SKILL.md).

Rule: derive it so a gap cannot compile; else publish it so consumers read; else refuse the copy
with a check. Watch the guard fail once before trusting it. A fallback list is the second copy.

## 6. Every exchange ends in an explicit typed terminal state; nothing is read from prose or silence

Example: a service closed requests by reading a finished turn's text or its silence, so one
request closed on "Final reply will follow" and the real answer was discarded. The fix is a typed
field in which the worker says this is its final reply. Full treatment: [protocol-design](../../protocol-design/SKILL.md) and
[stateful-shapes](../../stateful-shapes/SKILL.md).

Rule: accepted and finished are separate typed signals; every wait has a visible end; a
classifier of free text is never an enforcement boundary (see 10).

## 7. The user's inputs survive any restart by identity, accepted exactly once

Example: attachment uploads retried across a restart with a stable id, but the message send itself
did not, so a routine update could turn a user's message into a failed send. Stripe's idempotency keys and AWS REL04-BP04 ("make mutating operations
idempotent") are the same rule.

Rule: every human entrance retains the exact input and a stable id before transmission, retries
bounded transient failures with the same id, and never mints a new id on retry. Inventory every
entrance, not the one you tested.

## 8. A durable obligation has one owner, a stated stop condition and a recovery event

Example: hiding a view was conflated with stopping the work, risking dropped drafts and duplicate
resubmission; a request held "for a decision" with no release path stranded work; a repair
supervisor restarted without end because its attempt limit counted only the failure class it was
built for.

Rule: for each background activity: owner, what admits it, what ends it, what wakes a retry,
bounded across every failure class including the supervisor's own bugs.

## 9. Pending, success and failure are three distinguishable facts, and detection needs a subscriber

Example: an outage notice was delivered through the exhausted resource that caused the outage, so
it died with it; a sync alarm fired correctly into a channel nobody subscribed to. Ask where an
alarm is piped and who receives it. Full treatment:
[application-observability](../../application-observability/SKILL.md).

Rule: a failure path never depends on the resource that failed; every attempt records how it
ended; an alarm has a named receiver; "no data" is not "no failure".

## 10. Enforce at the boundary that can see the effect; a heuristic in front of every action is not a boundary

Example: a shell-text classifier meant to stop agents from building wait loops refused a
deliberate diagnostic and a read-only process listing, while the runtime already handled the
thing it guarded against; separately, an invariant enforced in callers instead of the lowest
boundary let deleted rows keep blocking new ownership. An ever-growing list of checks for every
command slows every agent down.

Rule: a pre-action check must name the pre-effect invariant it protects and the material
irreversible or conflicting consequence, and why no stronger boundary (the database, Git, the
credential boundary, the launch API) can refuse it. Everything else is advice or observation.

## 11. Minimum sufficient system: every part maps to a requirement, an invariant or an observed failure

Example: a fenced blue/green coordinator handoff with a socket broker was withdrawn once the
restart was measured as short and the client already had a restart retry policy.
[review-gates](../../review-gates/SKILL.md) owns the review side.

Rule: a comparative decision case names the objective, the smallest candidate, the upstream or
native path, doing nothing, and the measured evidence that separates them. Unmeasured benefits
are hypotheses.

## 12. "Done" means exercised on the real system; "Confirmed" means a citation someone checked

Example: "shipped" meant a compiled build and an untested change. In one design review, every
citation in the first design was re-verified: two paths were wrong and three documentation claims
were third-party write-ups mislabelled as official. Self-attestation is not validation.

Rule: a design cites `path:line`, a fetched URL or a pinned source commit for each claim and
labels the rest Likely or Unknown with the probe that would settle it; acceptance is a check on
the real system the user can see, never a unit test written to pass.
