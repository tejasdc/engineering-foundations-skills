# Exchange shapes

Choose by the failure you must prevent, not by what is easiest to code.

## Command / request (with a reply)
The sender asks the receiver to perform one bounded operation on one object and wants to know
what happened. The message names the operation (from the receiver's closed vocabulary) and the
object; the receiver answers with a terminal outcome.
- Right when: the sender knows exactly what should happen and someone is waiting on the result.
- Wrong when: many independent consumers each decide for themselves (use an event).
- Needs: an ID for deduplication, a return path, terminal states including refused and failed,
  and a rule for what happens while the receiver is away.

## Event notification (thin)
"Something happened" with minimal payload; receivers call back for details.
- Right when: consumers are many, details are large or sensitive, and the source is reliably
  reachable when they call back.
- Wrong when: the receiver must act while the source may be unreachable, or when the message
  omits the one fact the receiver needs so that it must re-read *everything* to find it. A
  notification that names nothing is a refresh signal in disguise (Fowler).

## Event-carried state transfer
The event carries the changed object's data, enough for the receiver to act without calling back.
- Right when: the receiver must act independently of the source's availability.
- Scope: carry **the changed object**, not a catalogue. Carrying every object is replication.

## Anti-entropy reconciliation (compare full state, repair differences)
Each side periodically compares its whole state with the other's and repairs differences.
- Right when: replicas are **meant to be identical** and delivery cannot be guaranteed, so a
  silently missed event must still converge (e.g. a code repository both machines must run).
- Wrong as a delivery mechanism, and wrong between parties that are **deliberately different**:
  every intended difference reads as "missing", which forces a mass copy (a backfill nobody
  asked for) or an ever-growing exception list. Its cost grows with total state, not with change.
- Legitimate as a backstop behind an event protocol — but only when the sets are meant to match.

## Hard and soft rules
Summer of Protocols frames a protocol as "an engineered argument" that manages a named tension
and manufactures certainty ("hardness") others can build on. Decide per rule how hard it is:
the operation vocabulary across a trust boundary should be hard (closed, refused otherwise);
retry spacing can be soft. Prefer a natural source of hardness — the receiver's own code, the
filesystem's "already exists" — over an invented rule nothing enforces.

## Decision questions
1. Does the sender know the specific change? → carry it.
2. Are the two sides meant to hold the same set? No → never compare sets.
3. Must the sender learn the outcome? → command with reply and terminal states.
4. Can the receiver be asleep? → the sender holds the message and retries; the receiver dedupes.
5. Does the receiver *do* something with side effects? → closed vocabulary, receiver-side checks.
