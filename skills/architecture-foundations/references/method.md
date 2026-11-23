# Design document template

Fill every section. "Unknown" is allowed only with the smallest probe that would settle it. Write
for the person returning to this later: problem and outcome first, in ordinary words, one concrete
example, who does what and what the user will see.

## 0. Requirement and profile
- The request, verbatim, with its source (message, date).
- Written priorities and decisions that bind it (ADRs or decision records, lines in the project's agent instructions).
- Operating profile: users, machines (which sleep), load, consequence of loss, reversibility,
  time available.
- The observed failure or acceptance criterion behind each new part. A part with none is cut.

## 1. Facts measured on the real system
A table: fact, evidence (`path:line`, log line, probe transcript, URL fetched on date). The
numbers that decide the design (restart time, cost per call, payload size, how long a dependency
takes) are measured, not estimated. Claims are labelled Confirmed (cited), Likely, or Unknown
(with its probe).

## 2. Inventory of what exists
- Lifetimes: each process, who spawns it, which supervisor owns it, what dies with it, what it
  holds open (pipes, sockets, locks).
- Durable state: each store, its owner, who writes, who reads, the migration path.
- Channels: each exchange between parties, its transport, what it retries, with what cadence as
  implemented, its reply path.
- Rollout: how a change reaches each part, including one that is already running.
- The upstream or native capability for each thing you are about to build (checked in official
  docs and the installed version, not a blog).

## 3. The invariant
One or two sentences that remove a class of cases. The state shape ([stateful-shapes](../../stateful-shapes/SKILL.md)) and the
exchange shape ([protocol-design](../../protocol-design/SKILL.md)) that follow from it. If the design has grown a branch per
reviewer comment, this section is wrong.

## 4. Architecture
- Parts, which machine each runs on, what each is for, how often it changes.
- What calls what (diagram).
- Where each fact lives, and what it is never stored in (one home each).
- Lifetimes: owner, admission, stop condition, recovery event, per background activity.

## 5. Protocols
Per exchange: service, environment assumptions (lost, duplicated, delayed, asleep, version
skew), vocabulary, fields with the correctness job of each, forbidden slots, state machine per
party with terminal states, idempotency key, who retries and when, expiry or visible unbounded
wait, how the sender learns the outcome, old peers.

## 6. Failure matrix
Failure, outcome that preserves the invariants. Include: restart of each part mid-operation; a
lost reply; a duplicate; the machine that sleeps; rollback; cleanup of old versions; the human
cancelling during a result.

## 7. The comparative decision case
Objective; candidates including the smallest one, the upstream/native path and doing nothing;
where each wins and loses; the measured evidence that separates them; unknowns as unknowns.
Say which candidate is recommended and why in one paragraph.

## 8. Observability
For each exchange and each lifetime: the signal that says which state it is in, where an
operator or agent reads it, and what the user sees. Pending, success and failure
distinguishable. Who receives each alarm.

## 9. Implementation plan
Dependency-ordered steps, each a complete delivery with: what it depends on, the acceptance
check on the real system, what the user will see, cost (comparative), risk and its mitigation.
A step is never a partial activation of one feature. Name the one-time migration constraints
(what cannot be moved, what must finish on the old path) explicitly.

## 10. Sources and changelog
The sources that changed the design and how; what changed from the previous draft and why;
disagreements that remain, with each side's evidence, under each author's name.
