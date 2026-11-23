# Sources

Every principle in this skill traces to one of these. Compiled September 2026; direct reads unless marked.

## 1. Summer of Protocols (summerofprotocols.com)

Read: https://summerofprotocols.com/intro-to-the-protocol-reader ; https://contraptions.venkateshrao.com/p/in-search-of-hardness ; https://summerofprotocols.com/research/dangerous-dating-protocols. (See end-of-doc note on essays found only as titles, not full text.)

1. A protocol is "an engineered argument," not an engineered agreement — unlike an API (one dominant actor's designed outcome), a protocol institutionalizes an ongoing negotiation between competing interests that never fully resolves. — intro-to-the-protocol-reader
2. A "tension" is a tradeoff plus a conflict — name the specific tension a protocol manages (speed vs. safety, openness vs. abuse-resistance) instead of pretending it's permanently solved. — intro-to-the-protocol-reader
3. Protocols aim to keep playing an infinite game — success is continuation and adaptability, not a one-time win; design to survive adversarial/unexpected use. — intro-to-the-protocol-reader
4. A successful protocol becomes invisible, letting participants coordinate "without thinking" — if every use needs conscious deliberation, it hasn't succeeded yet. — intro-to-the-protocol-reader
5. Hardness is the capacity to create certainty about the future — identify exactly what certainty the protocol manufactures (a fixed point others can build on). — in-search-of-hardness
6. Right-size hardness deliberately — over-hard breaks on any deviation, over-soft fails to coordinate anything; choose rigidity per-rule rather than defaulting to maximum strictness. — in-search-of-hardness
7. Prefer a natural source of hardness (crypto proof, physical constraint, legal contract) over inventing artificial rigidity with no real enforcement behind it. — in-search-of-hardness
8. Protocols fail in nameable ways: low-friction exit with no accountability (ghosting), a template that breeds generic/low-quality behavior, opaque algorithmic gatekeeping, and optimizing an easily-gamed proxy metric instead of the real goal. — dangerous-dating-protocols

Not independently fetched: the anthology's essays on protocol "tempo"/temporality and on protocol evolution (from workplace-safety history) were confirmed to exist only at title/abstract level via search indexing — no standalone URL was found. Don't attribute a "ratcheting" framework to this source beyond what's above; it wasn't confirmed in fetched text.

---

## 2. Classic networking / distributed-systems protocol design

### Saltzer, Reed, Clark — "End-to-End Arguments in System Design"
Primary PDF (http://web.mit.edu/Saltzer/www/publications/endtoend/endtoend.pdf) fetched but unparsable (compressed PDF stream) — sourced instead via https://blog.acolyer.org/2014/11/14/end-to-end-arguments-in-system-design/, which quotes the original directly.

1. Put correctness-critical functions (encryption, duplicate suppression, sequencing) at the endpoints — only the application at the ends can verify them completely; a lower layer's version is inherently incomplete.
2. Lower layers don't need to be perfect, only helpful — partial reliability there is justified as a performance optimization, never as the source of correctness.
3. The careful-file-transfer example: compute a checksum once at the source, transfer, verify at the destination — simpler than, and still necessary regardless of, per-hop error correction.
4. Don't duplicate a guarantee across layers "just in case" — if the endpoint must check anyway for correctness, an identical mid-network check is pure overhead.
5. Name who is responsible for a guarantee and whether they can trust the entities they depend on for it.
6. Exception: shared/contended resources. Congestion control must live in the network layer since it manages a resource shared across endpoints that can't be trusted to self-regulate.

### Postel's robustness principle
RFC 760: "an implementation should be conservative in its sending behavior, and liberal in its receiving behavior"; reworded in RFC 1122 as "Be liberal in what you accept, and conservative in what you send." (Wording via https://en.wikipedia.org/wiki/Robustness_principle; RFC 1122 text not independently re-fetched this session.)

### RFC 9413 — "Maintaining Robust Protocols" (critique of Postel)
https://www.rfc-editor.org/rfc/rfc9413.html

1. Tolerating unexpected/undefined input causes "protocol decay" — implementations add ad hoc workarounds that become de facto, undocumented parts of the protocol.
2. Design extensibility explicitly, with clear rules for new elements — don't rely on implementations gracefully ignoring the unknown; retrofitting extensibility later is hard.
3. Early implementations have a stronger obligation to follow spec exactly — their quirks become permanent constraints every later implementation must tolerate.
4. Prefer active spec maintenance over passive tolerance of drift — resolve divergence through a standards discussion, not by quietly accepting it forever.
5. "Virtuous intolerance": generate a fatal error on an unspecified condition instead of silent recovery, so the fault gets noticed — but only works if applied consistently and broadly.
6. Deliberate exclusion of a known-broken implementation is a legitimate tool to protect the ecosystem, as long as its impact is understood beforehand.

### RFC 3117 — "On the Design of Application Protocols"
https://www.rfc-editor.org/rfc/rfc3117.html

1. Prefer octet-counted framing over delimiter/stuffing, but still add a trailer/consistency check to catch implementation bugs.
2. Support asynchrony/multiplexing within a connection when the application has independent operations in flight, rather than forcing strict serialization.
3. Reuse an existing auth framework (e.g. SASL) instead of inventing one; keep one identity per session.
4. Choose familiar, established encodings (MIME/XML-like) over theoretically optimal but unfamiliar ones — the "social infrastructure" cost usually dominates the format cost.
5. Use structured reply/status codes with a separate human-readable diagnostic string.
6. Provide simple extension hooks; minimize how much extensibility is actually needed by getting the core right first.
7. "Easy things should be easy, hard things should be harder" — pain proportional to gain; don't let rare complex cases complicate the common path.
8. Add explicit consistency checks despite Postel's principle — leniency alone lets buggy new implementations go undetected.

### Holzmann — "Design and Validation of Computer Protocols"
http://user.informatik.uni-goettingen.de/~fu/design_principles.htm (full rule list); cross-checked, partially, against http://spinroot.com/gerard/popd.html.

Five elements every protocol must define: (1) the service it provides, (2) the environment assumptions (what can be lost/duplicated/reordered/delayed/corrupted), (3) the vocabulary of messages, (4) the encoding of each message, (5) the procedure rules (state machine) keeping exchanges consistent.

Ten design rules: (1) enumerate all design criteria/requirements/constraints up front; (2) define the service at every abstraction level before choosing internal structure; (3) design external (black-box) behavior before internal behavior; (4) keep it simple — fancy protocols are buggier and harder to verify; (5) don't connect what is independent — separate orthogonal concerns; (6) avoid arbitrary restrictions, design for a problem class not one instance; (7) prototype and verify against the design criteria before implementing; (8) implement, measure, optimize only as needed; (9) check the optimized implementation is still equivalent to the verified high-level design; (10) don't skip rules 1–7.

---

## 3. Messaging patterns

### Transactional outbox — https://microservices.io/patterns/data/transactional-outbox.html
1. Write the outgoing message to the database in the same transaction as the business-state change — the only way to get atomicity without a distributed transaction.
2. A separate relay reads the outbox and publishes to the broker, either by polling the table or tailing the transaction/commit log.
3. The relay guarantees at-least-once, not exactly-once — it can crash after publishing but before marking the row done.
4. This makes idempotent consumers mandatory, not optional, since brokers themselves also commonly redeliver.
5. Preserve per-aggregate ordering even with multiple service instances writing concurrently.
6. Event Sourcing is a named alternative that solves the same atomicity problem by making the event log itself the source of truth.

### Idempotent receiver — https://www.enterpriseintegrationpatterns.com/patterns/messaging/IdempotentReceiver.html
1. At-least-once delivery means "sent once" does not imply "received once" — assume duplicates arrive even if only one copy was sent.
2. Idempotency is f(x) = f(f(x)) — applying an operation twice must equal applying it once.
3. Two paths to idempotency: explicit de-duplication (detect and drop repeats), or message semantics that are naturally idempotent regardless of replay count (e.g. "set balance to $100" vs. "add $10").
4. This is the necessary complement to any guaranteed-delivery pattern — you can't claim reliable delivery without handling the duplicates reliability produces.

### Event Notification vs. Event-Carried State Transfer — https://martinfowler.com/articles/201701-event-driven.html
1. Event Notification: a thin "something happened" signal, no expected reply, minimal payload — cheap, but interested parties must call back for details, and the overall flow across many notifications is hard to see/debug.
2. Event-Carried State Transfer: the event carries enough data for the consumer's own copy — consumers become independent of the source's availability and stop generating callback load, at the cost of duplicated data and real state-maintenance logic.
3. State transfer survives source outages; notification does not, since the consumer must reach the source to act.
4. Event Sourcing and CQRS are separate, composable ideas: event sourcing makes the append-only log the system of record; CQRS just splits read/write models and doesn't require events at all.

### Store-and-forward
- SMTP (RFC 5321, §4.5.4.1/§4.2.5, via search summary — RFC text not independently re-fetched): "mail that cannot be transmitted immediately MUST be queued and periodically retried by the sender," which "retains responsibility for delivery" until success or an explicit bounce.
- DTN (RFC 4838, https://www.rfc-editor.org/rfc/rfc4838): for environments where an end-to-end path may never exist at any instant — store data inside the network (not just at the edges); use variable-length whole "bundles" instead of streams so scheduling is per self-contained unit; use flexible URI-based naming so addressing schemes can coexist; layer hop-by-hop custody transfer with an optional end-to-end ack on top.

### Acknowledgements, terminal states, expiry/TTL
Synthesized across the sources above (no single additional URL): a message needs an explicit terminal state — delivered/failed/expired — because "sent" is not a state the sender can trust on its own; a pending exchange with no expiry can wait forever for a condition that never occurs, silently consuming resources or blocking a dependent operation.

### Capability-based narrow commands vs. remote script execution — https://man.openbsd.org/sshd
1. A forced command (`command="..."` in authorized_keys) replaces whatever the client asked to run — the caller's own request is ignored and only exposed, optionally, via `SSH_ORIGINAL_COMMAND`.
2. Restrictions compose into a minimal-privilege channel: `restrict` (disables port/agent/X11 forwarding, PTY allocation, `~/.ssh/rc`) plus a forced `command=` gives a key exactly one capability.
3. Prefer a narrow, named command over general script/shell execution as the granted capability — the operational form of least authority, bounding a leaked key's blast radius.
4. A certificate-embedded forced command must match the local `command=` option exactly, so a certificate can't silently widen what a key is allowed to do.

---

## 4. FIPA ACL Request Interaction Protocol (brief)

Confidence note: the canonical spec (http://www.fipa.org/specs/fipa00026/SC00026H.html) is dead — the domain now resolves to unrelated third-party content — and PDF mirrors found could not be parsed into text by the fetch tool. The points below come from a search snippet that appears to quote the live spec directly, corroborated by JADE's docs for the same protocol (https://jade.tilab.com/doc/api/jade/proto/AchieveREInitiator.html). Treat this section as lower-confidence than the rest.

1. The initiator is never guaranteed a "yes" — a `request` can legitimately get back `refuse`; agreement is a distinct, optional step, not automatic.
2. Commitment and completion are separate messages — `agree` (accepting the work) is reported separately from `inform-done`/`inform-result`/`failure` (finishing it), giving a long-running action an explicit "started" signal distinct from "done."
3. Failure is a first-class outcome, not silence — an explicit `failure` message covers "agreed but then couldn't complete," rather than leaving the initiator to infer failure from a timeout.
4. The same shape generalizes across a protocol family (FIPA-Request, FIPA-Query, FIPA-Request-When, …) — all variants of "initiator asks for a rational effect, participant reports whether/when it was achieved," implemented generically rather than per-request-type.
5. Two-phase acknowledgement is what makes async request/response viable across unreliable agents — without a separate `agree` step, the initiator can't distinguish "never arrived," "arrived but rejected," and "arrived, accepted, still working."

---

## Sources that could not be fully read

- **FIPA spec canonical page** (http://www.fipa.org/specs/fipa00026/SC00026H.html) — domain no longer serves the specification; not read.
- **FIPA spec PDF mirror** (https://www.eecis.udel.edu/~decker/courses/886f04/pubs/FIPA-ips.pdf) — text could not be extracted from the compressed PDF; not read directly.
- **Saltzer/Reed/Clark original PDF** (http://web.mit.edu/Saltzer/www/publications/endtoend/endtoend.pdf) — text could not be extracted; a secondary summary with direct quotes was used instead.
- **Wayback copies of the dead FIPA page** — not retrieved.
- **unicamp.br FIPA mirror** (https://www.dca.fee.unicamp.br/projects/sapiens/...) — host unreachable at the time.
- **Summer of Protocols essays on "tempo"/temporality and on protocol evolution** (inside the 358-page Protocol Reader anthology) — seen only at title/abstract level; no standalone URL found.
- **RFC 1122 and RFC 5321 primary texts** — not re-read directly; wording above comes via secondary sources. Reliable but secondary.
