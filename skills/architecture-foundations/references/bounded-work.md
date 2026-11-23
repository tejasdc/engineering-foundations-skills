# Bound the work the person waits for

Use this when a feature reads a collection, refreshes after an event, replays retained output,
or adds a process lifetime. The acceptance property is that opening one page, receiving one
input and applying one change do not require work proportional to unrelated retained history.

Before implementing, identify four things in the owning design or executable contract:

- **Growth:** what can grow (records, bytes per record, open items, concurrent executions,
  replayed bytes, fan-out, refresh frequency), and the bounded access path for each interactive
  operation. Measure indexed seeks and bytes as well as returned rows; an index is not a bound.
- **Writers:** every authority that can change a displayed fact and how its durable change
  reaches the prepared reader. Include deletion, project movement, time-based transitions,
  restart and expired cursors. A rebuildable projection is not a second canonical writer.
- **Lifetime:** which process holds each buffer, child, subscription and cache; what releases
  it; what survives a coordinator restart. Reparse new frames, not accumulated output.
- **Refusal and proof:** the API refuses an undeclared interactive reader, and the release
  runs its actual reader under representative cardinality, large values and unrelated-history
  growth. A cost cap after allocating a full body is not prevention. Put exceptional control
  reads in an exact named registry; a broad prefix exemption defeats the boundary.

One event invalidates its actual dependants. A cached page plus a cursor is useful only if
the client, intermediary and owner accept the same cursor. An expired cursor resets to a
bounded page explicitly; an ordinary error must not silently request the entire collection.
An incomplete projection is unknown or catching up, never evidence that the collection is empty.

Test the complete serving path: indexed ID lookup **and** subsequent joins, JSON parsing,
attribution and response construction. Seed the largest realistic record, not just many tiny
rows. For cached/offline behavior, first visit through the real app, then close/reopen it while
the owner is unavailable. Do not inject a cache the product never writes. Keep custody,
owner acceptance, provider observation and paint as separate timings and states.

Rollout also has a growth/lifetime boundary. Exercise the updater already installed against
the candidate declaration; a new builder cannot prove the old builder will package its new
workers. Preserve older clients' pending actions and expose an explicit compatibility boundary.
Never silently truncate a supposedly complete result to satisfy a budget; provide continuation
or exact detail on demand.

## Evidence and first wrong explanations

In one notes app, the client retained a browser copy of a feed and sent its change cursor; an
intermediary rejected that cursor once it outgrew a length limit, and the client silently fell
back to reading every receipt. The fix was not another index. The first report wrongly inferred
that the page never sent the cursor from the absence of requests at the origin. Trace all three
sides: client, intermediary, owner.

In a hosted-execution system, retained runs grew longer and live output reparsed all previously
replayed bytes. That defect was established; a rumored multi-gigabyte allocator was not.
Resource and custody correctness must be exercised under load, not inferred from two short
successful restarts. During implementation, a small browser fixture passed while the real app
could not reopen the cached conversation; an ID-only fixture passed while the route still parsed
full multi-megabyte bodies. Both illustrate why the complete consumer is the oracle.

This skill is guidance, not enforcement: pair it with a reader registry, a runtime refusal for
undeclared interactive readers and a growth gate in the release, and check rollout and live
acceptance separately before describing those guards as installed.
