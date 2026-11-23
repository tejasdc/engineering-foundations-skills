# Incidents that produced these rules

All incidents are anonymized; the technical lessons are what matter.

## Ownership, recipient identity and subscription confused
A team wanted a service to read one shared secret store. The explanation conflated the resource
owner, the receiving identity and the plan entitlement, and was written before reading the
vendor's current documentation. Correction: read the vendor's current official documentation
first, keep those three separate, and say what reaches the receiver and what stays private.
Rule 2 and Step 1. This is research guidance, not a runtime check.

## A sender's Queue default delayed messages to a running agent
Messages sent to a running agent waited as "Queue" because the sending route supplied that choice
itself and the receiver honored it. An earlier fix had changed only senders that omitted the
choice, so old routes and other callers could still defeat live delivery. The receiver now derives
the default from whether the run is accepting input, and clients mark a deliberate Queue
separately. This is a protocol default enforced by the receiver, not an instruction for senders to
remember. Rule 10.

## Telling a peer what changed instead of making it diff everything
Asked for one way to create a project that also appears on a second machine, the first design had
the creator send a thin "your project list may be stale" notice. The other machine then fetched the
whole list and copied whatever it lacked, on every notice, startup and reconnect. The two machines
hold different sets on purpose, so every comparison read as "missing". The design also invented a
weaker nudge beside an existing durable request protocol that already queues and retries while the
second machine sleeps, and located its safety in "the receiving machine's rules" without naming the
program. No protocol source had been consulted. Better: tell the receiver what was added, reuse the
durable channel, name where each check runs, and ground the design in published patterns.
Rules 1, 2, 3, 4, 7.

## Requests closed by reading prose
A session manager closed agent-to-agent requests by interpreting a finished turn's text or silence,
and discarded a real answer that arrived after a "final reply will follow" message. Fixed with
typed reply commands carrying dispositions and explicit states: the agent says "this is my final
reply" as a function call rather than the system guessing from wording. Rule 5. Detail:
[stateful-shapes guideline 13](../../stateful-shapes/guidelines.md).

## A change notice treated as "re-read everything"
An inbox re-read its whole event ledger on every change instead of reading from its cursor or the
one record an event named. Rule 1. Detail: [stateful-shapes guideline 12](../../stateful-shapes/guidelines.md).
