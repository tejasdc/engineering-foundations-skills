# Protocol design template

Fill every section. "Unknown" is allowed only with the smallest check that would settle it.

## 1. Service
One sentence: what the protocol achieves for whom. Include the scope boundary (which objects,
which not — e.g. "only projects someone explicitly created or asked to move").

## 2. Parties and existing channels
- Parties, and which program on which machine speaks for each.
- Every existing channel between them (transport, authentication, what it carries).
- Which existing operations are durable (kept and resent) and which fail fast.
- Retry cadence **as implemented** (cite the code), and what wakes a retry.
- The existing protocol this one is a sibling of, and what it reuses from it.

## 2b. The originating action
If the protocol announces something one party does first (create, delete, publish), describe that
action too: its steps, which are external, how a crash between them is recovered, and the natural
lock if both parties can do it.

## 3. Environment assumptions
For this channel: can a message be lost, duplicated, reordered, delayed? Can a party be asleep
for days? Can the two parties run different code versions? Can the channel's secret be used by
something other than the intended sender?

## 4. Vocabulary
The closed list of message kinds. For each: meaning, who sends it, what the receiver may do.

## 5. Message fields
A table: field — why it is needed — who produces it — how the receiver verifies it.
Then list the **forbidden slots**: what the message deliberately cannot carry (script, command,
path, branch, free text instructions) and why.

## 6. Sender state machine
States, the signal that moves each, and the terminal states. Where the outgoing record is written
(same write as the change it announces?). What happens if the peer is unreachable. Cancellation.

## 7. Receiver state machine
Accepted → checks → action → terminal state. Deduplication by ID. The "already true" path. What
is never done (overwrite, delete, run repository code, start an agent).

## 8. Authority and enforcement
- Which program, on which machine, performs each check.
- The fixed action the receiver runs (exact command shape, no shell).
- Worst case if the channel secret leaked, compared with what it already allows.

## 9. How it ends and how the sender learns
Push of the result plus pull on the sender's wake, so a lost push is never a lost result. Where
the result lands (the asking conversation). Who the human hears from, and when (usually only on
refused/failed). Expiry, if any, and why.

## 10. Offline behaviour
Step by step: the receiver is asleep when the message is created; what the sender shows; when the
next attempt happens; what happens on the receiver's first contact; the note after a long wait.

## 11. Evolution and old peers
What an older receiver answers for an unknown operation, and how the sender classifies it (a clear
refusal naming the needed update, not a retry loop). How a new kind is added later.

## 12. Human surface
The command a person runs, what it prints, how to check status, what they see if something fails.

## 13. What it replaces, patterns used, open unknowns
