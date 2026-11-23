# Patterns (with sources in sources.md)

## Transactional outbox
Write the outgoing message in the same database transaction as the change it announces; a relay
sends it later. A crash can neither lose the message nor send it for a change that did not commit.
Delivery is at-least-once, so the receiver must deduplicate. (microservices.io)

## Idempotent receiver
"Sent once" does not mean "received once". The receiver keys on the message ID and returns the
existing result for a repeat, or the operation is naturally idempotent ("ensure X exists", not
"create X"). (Enterprise Integration Patterns)

## Store-and-forward
When the receiver is unreachable the sender queues the message and retries, keeping responsibility
until delivery or an explicit refusal (SMTP). For long disconnections, whole self-contained
messages are held rather than streams (DTN, RFC 4838).

## FIPA request
The initiator's request can be refused; "agree" (accepted) and "inform-done"/"failure" (finished)
are separate signals; failure is a message, not silence. Without the split the initiator cannot
tell "never arrived", "rejected" and "still working" apart.

## End-to-end argument
Checks that correctness depends on belong at the endpoint that can fully verify them; lower
layers may help but cannot be trusted as the guarantee. Put the receiver's safety checks in the
receiver's handler, on its own disk. (Saltzer, Reed & Clark)

## Least authority / forced commands
Grant one named operation instead of general execution: an SSH key with `command=` can run only
that command whatever the client asks. Across machines: a closed vocabulary of operations, each
with a fixed action. (OpenBSD sshd)

## Strict parsing
Silently tolerating unexpected input causes protocol decay; fail visibly on an unknown kind or
malformed field. An old peer that does not know a new operation must produce a distinguishable
refusal. (RFC 9413; RFC 3117 structured status codes with human text)

## Keep it simple, separate what is independent
Define external behaviour before internals; do not connect independent concerns; design for the
class of problem, not one instance. (Holzmann's rules)

---

# Anti-patterns
- A thin "something changed, re-read everything" notice when the sender knows what changed.
- Comparing full state between parties that are deliberately different.
- A new weaker mechanism beside an existing durable one.
- Sending a message and updating local state as two separate, non-atomic steps.
- Assuming "sent once" means "received once".
- A generic "run this" capability where one named operation would do.
- Treating "started" and "finished" as one signal; treating silence or prose as an answer.
- Locating a safety guarantee in the sender's tool or the transport.
- Silent tolerance of unknown input, including an old peer's "unknown route".
- A wait with no end and no visible state.
