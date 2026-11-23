---
name: protocol-design
description: Use BEFORE designing how two parties exchange anything — two machines, two services, two agents, a client and its backend — and before adding a message, event, notification, sync, webhook, queue, retry or "tell the other side" step. Triggers on "sync between devices", "what exactly crosses between the two sides", "what happens when the other side is offline", "where does the receiving side enforce this", "design the protocol", "how does it end", "how long do we wait", "duplicate messages", "how much of an existing account or plan can we reuse". Covers choosing the exchange shape (command, event, state transfer, reconciliation), reusing the channel that exists, the minimum message and why each field, a state machine per party with terminal states, offline and duplicate delivery, what the receiver may do and where that check physically runs, how the sender learns the outcome, old peers, and how to present the design. Do NOT use for the shape of state inside one system (use stateful-shapes) or for API route style.
---

# Protocol design

A protocol is the agreement two parties follow when they **share no memory**: everything one
knows about the other arrives in a message, may arrive twice, late, or not at all. Designing one
means deciding what crosses, who holds responsibility at each moment, and how every exchange
ends. It is not "add an endpoint".

The five things every protocol must define (Holzmann): the **service** it provides, its
**environment assumptions** (lost, duplicated, reordered, delayed, a party asleep), its
**vocabulary** of messages, their **encoding**, and the **procedure rules** — the states. A design
that cannot state all five is not finished.

## Critical rules (each one came from a real failure)

1. **Send the fact, not a hint to go look.** If the sender knows what happened, the message
   carries that one thing. "Your view may be stale, re-read everything" forces the receiver to
   guess (Fowler, event notification vs event-carried state transfer). Full-state comparison is a
   *repair* tool for copies that are **meant to be identical**; between parties that are
   deliberately different it turns every difference into a false "missing". Incidents:
   `references/incidents.md` → "A change notice treated as 're-read everything'" and "Telling a peer what changed instead of making it diff everything".
2. **Reuse the channel and protocol that already exist.** Inventory what already travels between
   these parties before inventing anything. A new operation is a **typed sibling** of an existing
   durable protocol, never a weaker cousin beside it (a "nudge" beside a queue that already
   retries). For a third-party provider's sharing feature, establish its native capability from current
   official documentation first, and keep the resource owner, the receiving identity and the
   plan entitlement separate. State what reaches the receiver and what remains private before
   proposing infrastructure (see Step 1).
3. **Every field earns its place, and some slots must not exist.** Justify each field by the
   correctness job it does (identity, deduplication, the one object acted on, return address).
   Across a trust boundary there is no field that can hold a script, a shell command, a path or a
   free-form instruction. The receiver derives paths itself.
4. **The receiver's own code is the guarantee.** Only the endpoint can verify what it is about to
   do (end-to-end argument). The sender's command-line tool is a courtesy check anyone can bypass;
   a shared secret on the link usually opens everything. Say, by name, **which program on which
   machine** performs each check.
5. **Every exchange ends in an explicit terminal state**, produced by a typed signal, never read
   from prose or silence. Refused and failed are answers. "Accepted" and "finished" are separate
   facts (FIPA request). A wait with no end must be visible, not silent.
6. **Assume duplicates and sleep.** Record the outgoing message in the same write as the change it
   announces (transactional outbox); the sender keeps responsibility and retries (store-and-
   forward); the receiver keys on a message ID so a resend is harmless (idempotent receiver).
   Name what wakes a retry — a timer, the peer's next contact — and check the code actually does it.
7. **The receiver builds addresses itself.** Send names, never locations: an identifier the
   receiver turns into a URL or path with parts it fixes itself (its own owner, its own root). A
   field shaped like a URL or path is a slot for misuse even when validated.
8. **Scope is the human's.** Never widen "this object" into "every object" (backfill, mirror
   everything) unless asked. An existing object moves on an explicit request.
9. **A contract that makes a party say a thing twice is the bug.** Change what the protocol asks
   for so each thing is said once; never filter, hide or suppress repeated content by comparing
   words, hashes or bytes. A duplicate that reaches the human stays visible, because it marks a
   protocol fault to fix. Rule 6's idempotency keys on message identity, not content. Example: a
   service posted an answer and then a separate closing message. Suppressing closers by wording
   would miss variants; the fix is for the receiving side to record which turn posted where and
   present the closing result only when that turn did not already post there. The check uses
   causal identity, not similar wording.
10. **The receiving owner chooses a message's default delivery.** Inventory every sender of the
    same addressed message, including old clients, agents and service returns. An omitted choice
    means "join the live run if it can accept this message"; only a separately recorded, deliberate
    "queue behind the current run" choice may make it wait while that run is accepting input. A sender's preselected queue
    field is not proof of a human choice. Verify a real send and the receiving owner's retained
    decision in one run, then repeat through a peer. See "A sender's Queue default" in
    `references/incidents.md`.
11. **A person's words keep their role across every handoff.** Text or speech captured for an
    ordinary agent session remains the exact human input in stored history, provider input,
    steering, retry and interrupted-turn recovery. Put application identity, tool instructions and
    continuity metadata in a separate application-context channel; a separate storage column alone is not
    enough if provider preparation concatenates it back into the user's message. Check the
    receiver's old-version capability before capture when an unsupported field would become a
    terminal refusal. Verify the agent's action route on every selected machine and project,
    rather than inferring parity from a shared session label.

## Workflow

### Step 0 — Name the exchange shape
Pick from `references/exchange-shapes.md` by asking: does the sender know the specific change?
Are the two sides supposed to hold the same set, or different sets on purpose? Must the sender
learn an outcome? Typical answers:

| Situation | Shape |
| --- | --- |
| Sender wants the receiver to do one bounded thing and learn the result | **Command / request** with a reply |
| Something happened; receivers act on their own | **Event**, carrying the changed object |
| Receiver needs a copy of one object's data | **Event-carried state transfer** of that object |
| Replicas meant to be identical may have silently drifted | **Anti-entropy reconciliation** (backstop only) |

### Step 1 — Inventory what exists
Before designing, read the code and write down every channel between the parties: transport,
authentication, what each carries, which operations are durable (kept and resent while the peer
is away) and which fail fast, retry cadence *as implemented* (docs drift — read the timer), and
the existing reply path. Decide which existing protocol the new operation is a sibling of.

For third-party sharing, include plan eligibility, recipient requirements, machine access and
the actual access boundary, from current documentation rather than remembered older plan rules.
Separate verified capability from an untested setup, and missing documentation from proof that
the capability does not exist.

### Step 2 — Fill the design template
Use `references/design-template.md`. It walks the five elements plus: minimum fields with a
reason each, forbidden slots, sender and receiver state machines, idempotency key, offline
behaviour, expiry, how the outcome returns, who (if anyone) the human hears from, version skew.

### Step 3 — Place authority
List what the receiver is allowed to do on receipt as a **closed vocabulary** of named operations,
each with one handler and a fixed action (e.g. one git command with fixed arguments, no shell).
Adding a capability later means adding a named operation, never a generic "run this". Narrow
capability beats general execution (SSH forced commands). State the worst case if the channel's
secret leaked, compared with what that secret can already do.

If the receiver needs a semantic judgment among those fixed operations, a rules engine or a
constrained model can make it, provided its typed judgment stays inside the receiver's checked
vocabulary; it never grants a new operation.

Make the fixed action itself inert: know every way that tool can run code or reach elsewhere
and switch each off (see references/safe-actions.md for git clone). Do the action in a temporary
place and move it into its final place last, so a half-finished result never looks finished.

### Step 4 — Define "done" by the receiver's own predicate
"Done" is the receiver's observable fact that the goal holds (e.g. "a session can start in that
folder"), not "the command exited 0". Include the harmless path where the goal already holds
(someone did it by hand) — that keeps manual fallbacks from conflicting with the protocol.

### Step 4b — Settle the awkward cases (references/edge-cases.md)
Expiry versus a visible unbounded wait; a peer too old to understand the message (park and retry
slowly with one notice — the message is still valid); a transient failure after acceptance (a
resend retries it; a permanent one stays final); both sides able to originate (find the natural
lock, e.g. the name already existing upstream); a broader existing channel that already allows
more (state it honestly as a separate decision, do not silently widen or narrow scope).

### Step 5 — Check against the questions
Walk `references/checklist.md` (20 questions). Any unanswered question is either answered or
written down as an explicit unknown with the smallest check that would settle it.

### Step 6 — Present it
For the person you build for: parts, which machine runs each, what calls what, what crosses the
wire and why, the states, what they see and when, how they run it themselves, what it replaces,
the patterns used with sources, and the open unknowns. Use plain language; avoid code identifiers where the audience is non-technical.

## References

- `references/exchange-shapes.md` — command, event notification, event-carried state, anti-entropy,
  request/reply; when each is right and wrong; hard vs soft rules.
- `references/design-template.md` — the fill-in design document.
- `references/checklist.md` — the 20 questions every design answers.
- `references/patterns.md` — outbox, idempotent receiver, store-and-forward, FIPA request,
  end-to-end, least authority, strict parsing; one paragraph each with the source.
- `references/safe-actions.md` — making a receiver's fixed action inert (git clone hardening).
- `references/edge-cases.md` — expiry, too-old peers, transient failures, symmetric parties, broader channels.
- `references/incidents.md` — anonymized failures that produced the rules above.
- `references/sources.md` — every source with URL, and what could not be read.

Related: [stateful-shapes](../stateful-shapes/SKILL.md) (the state *inside* each party: ordering, single writer, durable
workflows; guideline 13 is the request-reply obligation rule); [architecture-foundations](../architecture-foundations/SKILL.md) for the overall design method. This skill is the exchange
*between* parties.
