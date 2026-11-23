# Awkward cases every design settles explicitly

## Expiry or a visible unbounded wait
Give a message an expiry when acting late would be wrong or harmful (a time-bound action, a
decision that goes stale). Leave it unbounded when acting late is still exactly right (a project
created on Friday should still appear on Monday) — but never silent: the requester sees "waiting
for X" immediately, and the human gets one notice after a wait long enough to mean something is
wrong (a day, not minutes). One notice, not one per retry.

## When the change is not a database write
The outbox rule (record the message in the same write as the change) assumes one database. When
the change is external (a repository pushed to GitHub), make the creation step idempotent instead:
re-running it detects the finished external state and only records the missing message. Record the
message after the external step succeeds, never before it.

## Knowing what the peer supports
Do not guess capability from an error text. Each party publishes the operations it accepts in its
existing status answer; the sender checks before sending and treats "not listed" as needs-update.

## A peer too old to understand the message
Different from a refusal: the message is valid, the receiver's code is behind. Classify the old
peer's generic answer (e.g. unknown route) as "needs update", park the message, send one notice
naming the update to run, and retry on a slow cadence (hourly) so it completes by itself once the
peer is updated. This is not a hot retry loop; do not fail it.

## A transient failure after acceptance
The idempotent receiver returns its recorded result for a repeat — except when that result was a
transient failure (network, temporarily unavailable). Classify failures as transient or permanent
when recording; a resend of a transient failure re-attempts; a permanent failure or refusal is
final and a new attempt is a new request from the human or agent.

## Both sides can originate
When either party can create the object, find the natural lock that makes a race impossible to
lose silently — usually the upstream authority refusing a duplicate (the repository name already
exists). The loser gets a clear refusal. Do not invent a coordinator when one already exists.

## Message identity
Derive the ID from what makes the request unique (target, object, operation), so the same request
cannot be live twice; a genuine retry by a person after a final failure uses a fresh attempt.

## A broader channel already exists
If the new narrow operation rides a channel that already allows much more (an agent-starting
request, a shared admin secret, an SSH key), say so plainly: the new operation adds nothing a
leaked secret cannot already do, and the stated guarantee ("nothing an agent sends can run
anything else") holds only for the new operation. Narrowing the old channel is a separate
foundation decision for the human, not silent scope in this design.

## Where the human's command lives
The operation lives with the service that performs it; the human-facing command is a thin,
self-explanatory wrapper that lives in the project's shared tools directory, and is
usable by agents and the human alike.

## Values that differ per machine never travel
Anything whose correct value depends on the machine (a local folder path, a home directory, a symlink)
is computed by the receiver on arrival, never carried in the message or in the object being
copied. Example: a symlink to a machine-local folder was committed and then cloned onto another
machine where that folder does not exist. The fix kept the link out of the repository and let
each machine make its own.
