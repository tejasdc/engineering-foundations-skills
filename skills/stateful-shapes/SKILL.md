---
name: stateful-shapes
description: "Use when designing, changing or reviewing anything that keeps state over time - chat, messaging, queues, workflows, agent threads and handoffs, sync between devices or machines, session or lifecycle status, timers and progress, retries, single writer or actor/coordinator, state machine, XState, Temporal, Durable Objects or a sync engine - and when something stateful goes wrong: duplicated sends, answers that never arrive, dropped or stuck requests, work killed by a restart, stale status, progress that disappears and returns, changes lost after a save, notifications sent to the wrong recipient, skipped scheduled runs, protocol design for retries and waits. Not for copy, styling or formatting changes."
---

# Stateful Shapes

## Overview

Stateful complexity is not one thing. Mature systems fall into a small number of recognizable shapes, and the usual mistake is the wrong shape, not a forgotten retry. This skill names the shapes, the rules real incidents taught, and the threshold for a heavier library.

The skill helps most when a shape is turned into one traced question ("which entity owns this transition, and where does its event become durable?") and least when it is read and cited without changing anything. So it asks for a short written state card and a check that would fail, not a reading.

## First: is this worth a state review?

Name one of these in a sentence before going further: an observed failure (lost, duplicated, reordered, stale, regressed, stuck), or an invariant the change must keep. If you cannot, stop; this is not a stateful change. Copy, styling, formatting and pure presentation tweaks never need it.

State the operating profile next: for example one person, a few devices, two machines, unless the project says otherwise. Stripe- or Temporal-grade durability is a reference for the shape, not a bar to meet. Applying this vocabulary to cosmetic state (a table, several functions and a worker to remove an emoji) is over-engineering; that is why this gate and profile exist.

## The state card (write this; one screen)

Put it in the design, the plan or the review. It is this skill's deliverable.

1. **Entity** that owns ordering, and the **one writer** for it. If two writers touch one record, which fields each owns or how they merge ([guideline 6](./guidelines.md#6-if-many-code-paths-mutate-one-entity-make-one-coordinator-instead-of-more-rules)).
2. **Operation identity** that survives retries, refreshes and restarts: one per attempt or run, never a calendar date ([guidelines 4](./guidelines.md#4-do-not-use-one-status-field-as-workflow-history) and [5](./guidelines.md#5-add-idempotency-at-the-boundary-as-soon-as-retries-can-duplicate-effects)).
3. **Acceptance ladder**: recorded by us → admitted by the transport → saved durably by the receiver → shown to the person. For each success signal the code checks, the rung it proves ([guideline 16](./guidelines.md#16-a-success-signal-proves-only-what-it-measures)).
4. **Provisional and retained state**: what the screen shows before the durable answer, and how the two merge by identity ([guidelines 11](./guidelines.md#11-preserve-user-action-meaning-across-requests-and-refreshes) and [15](./guidelines.md#15-merge-progress-by-identity-before-aggregating)).
5. **Lifetime**: which process holds the work, what restarts it, and whether that is the thing updated most often ([guideline 14](./guidelines.md#14-separate-the-lifetime-of-what-does-the-work-from-the-lifetime-of-what-coordinates-it)).
6. **Old readers**: the older installed page, worker or peer that will meet the new record or message ([guideline 17](./guidelines.md#17-an-older-installed-reader-will-meet-every-new-record)).
7. **Failure trace** in five steps: before the effect, effect committed, response lost, refresh or restart, what the person sees.
8. **One falsifying sequence and the check that runs it** (a test, a fault injection, an installed-build check). A card without a check is context, not prevention.

For cross-machine withdrawal, trace two distinct facts: the origin accepted cancellation and the receiver revoked execution. A queued turn on a sleeping peer can stay unchanged while its origin is canceled, so the receiver must check the origin after reconnect or the origin must retry until revocation is acknowledged. Revoke unstarted work and its redundant cancellation instruction before a quota reset can claim either; leave running work to its running cancellation path. A notification about waiting work also checks the exact live origin and receiver turn before delivery.

For a task created on behalf of another continuing agent, record that agent as an interested destination and deliver a durable, idempotent status transition to it. Keep the first completion event distinct from a later review that repeats `done`; a repeated `done` must not rewrite the completion time. At the final boundary that produces a task-derived ask or attention item, current human completion must veto a repeat request even when no external receipt was saved; clear matching pending asks without clearing unrelated tasks. This veto must cover ordinary conversational follow-ups too, not only a structured attention list. A missing URL is an unavailable artifact, not proof that the human did not finish. Example: a user marked a task done through a widget; the planning agent never received the completion, and later reviews rewrote the displayed completion time. The task system owns transition delivery, first-completion preservation and the output veto; until those exist, the originating planner must check the task's status before follow-ups.

## A step's exit belongs to its completion rule, not its first child

When a repeated activity has a variable number of units, derive Next versus Return from the
remaining units in the retained activity. Never give each card or child the activity's close
callback as its advance callback. Keep explicit interruption separate from completion; map
ordinary navigation, notification navigation, visibility return and full document replacement
against the same retained progress before claiming the flow fixed. Exercise at least two units,
interrupt after the first, then finish the last while an already-open second surface observes.
Pictures of the next card and a stored completion are different evidence from that sequence.
The marked checking entrance must wire the same change-announcement port as production: observe
the actual announcement on the already-open page before crediting a passive update. An incidental
opening or visibility reread can otherwise make missing stream wiring look like a pass.

Example: a multi-unit step (two questions) exited after the first unit because the step's close action was passed to every card as its advance action. Earlier fixes checked pictures of the next card rather than advancing through the step. The repair enforces the exit rule in the shared path and retains a browser sequence (two units, interrupt after the first, finish the last) that fails without it.

### Accepted progress and permission to continue are the same boundary

When an accepted attempt satisfies one unit, enable its advance from that accepted fact.
Extra rehearsal can remain optional; it must not become an undisclosed second completion rule.
Show one progress measure for the retained step. An assertion that a wrong answer disables Next
is not acceptance when the requested outcome is to continue after an attempt.

A recurrence of the same flow showed credit at one of two, a competing zero-of-two counter, and
Next disabled until a correct repeat; an earlier repair had checked wrong-answer, then
correct-repeat, then Next and called it passing, so it never challenged the forced repeat against
the requested flow. A phone-only variant survived because only one surface (the review queue) was
filtered while another (Browse) was unobserved. Apply an excluded-platform rule to inspection, deep
links and retained sessions as well as starting an exercise, rather than treating pointer/hover as
proof of a keyboard.

## The seven shapes

| Shape | Where it occurs | Key invariant | Canonical team | Detail file |
| --- | --- | --- | --- | --- |
| Ordered event stream per entity | Chat rooms, issue histories, payments | One authoritative ordering boundary per stream | Slack / Discord / Telegram | [01](./shapes/01-ordered-event-stream.md) |
| Replicated collaborative document | Shared docs, design files, block graphs | Concurrent edits converge at the right merge unit | Figma / Notion / Yjs | [02](./shapes/02-collaborative-document.md) |
| Durable workflow / lifecycle execution | Payments, approvals, retries, timers | Progress survives crashes, waits, and retries | Temporal / Cadence / Stripe | [03](./shapes/03-durable-workflow.md) |
| Single coordinator / actor per entity | Rooms, carts, sessions, queue shards | Exactly one live mutation coordinator per entity | Durable Objects / Orleans / OTP | [04](./shapes/04-actor-per-entity.md) |
| Ephemeral awareness / presence | Typing, cursors, online state, visibility | Loss or delay must not corrupt durable history | Slack / Phoenix / Yjs | [05](./shapes/05-ephemeral-awareness.md) |
| Sync engine / local-first cache | Instant local reads with later reconciliation | Local state stays usable now and converges later | Linear | [06](./shapes/06-sync-engine.md) |
| Durable conversation / agent thread | Tool-using agents, resumable runs, human approval | Continuity lives in the thread, not the latest message | Anthropic / OpenAI / LangGraph | [07](./shapes/07-agent-thread.md) |

## How to use this skill

1. Pass the gate and state the profile.
2. Identify the entity and the failure mode; match one or more shapes (they compose). Read only the matching shape files and the guidelines the card points to.
3. Write the state card, then choose the lightest tool that honestly solves the pressure.
4. **When you cite this skill in a review or report**, name the guideline, the design or code choice it changed, and the check that fails without that choice. Shapes listed with no changed choice are background; say so.

## Decision: when to adopt a library

Use [library-thresholds.md](./library-thresholds.md). Do not reach for XState because you have many states; reach for it when the failure mode demands hierarchical concurrency. Do not reach for Temporal for a few labels; reach for it when progress must survive crashes, timers, or retries. This skill should never be the reason a heavy library is adopted.

## Common mistakes

- Picking a shape by state count rather than failure mode.
- Collapsing multiple shapes into one aggregate.
- Persisting ephemeral awareness as durable business state.
- Using a `status` column as a substitute for workflow history, or a hand-written status that outlives the work it describes.
- Retrying ambiguous operations without idempotency keys.
- Treating "the call returned success" as "the receiver has it" ([guideline 16](./guidelines.md#16-a-success-signal-proves-only-what-it-measures)).
- Shipping a new record kind with no check that an older open page can read or pass it through ([guideline 17](./guidelines.md#17-an-older-installed-reader-will-meet-every-new-record)).
- Assuming replicas can "just take the latest" without sequence or state-resolution rules.
- Buying CRDT or OT complexity before there are concurrent editable shared objects.
- Refetching a whole, growing collection whenever one item in it changes ([guideline 12](./guidelines.md#12-make-every-read-cost-what-changed-not-how-much-history-exists)).
- Subtracting provisional progress from a total that may exclude its entity ([guideline 15](./guidelines.md#15-merge-progress-by-identity-before-aggregating)).
- Applying Stripe-grade durability to cosmetic or single-user state.
