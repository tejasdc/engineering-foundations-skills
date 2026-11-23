# Practical guidelines

The first ten rules are compiled from the public sources linked under each rule. Later rules come from implementation experience.

## 1. Pick the entity that owns ordering before writing code

Rule: name the coordination key first.

Slack and Discord stay tractable because they make the ordering boundary explicit: channel first, everything else second. If the system cannot answer "who owns ordering for this entity?" it usually starts compensating with retries, timestamps, and after-the-fact repair.

Source teams: [Slack](https://slack.engineering/real-time-messaging/), [Discord](https://discord.com/blog/how-discord-stores-trillions-of-messages)

```ts
// Bad: two writers decide order independently.
saveMessage(roomId, message, Date.now());

// Good: one room stream assigns sequence.
stream.append(roomId, { kind: 'message.sent', message });
```

## 2. Separate durable history from ephemeral awareness

Rule: typing, presence, and cursors are not message history.

Slack, Phoenix Presence, and Yjs all split durable state from perishable awareness. This keeps loss, duplication, or delay in the awareness layer from corrupting the durable record.

Source teams: [Slack](https://slack.engineering/real-time-messaging/), [Phoenix Presence](https://hexdocs.pm/phoenix/presence.html), [Yjs Awareness](https://docs.yjs.dev/getting-started/adding-awareness)

```ts
// Bad: typing mutates durable history.
events.push({ kind: 'typing.started', userId });

// Good: typing is a TTL-based broadcast surface.
presence.heartbeat(roomId, userId, { typing: true });
```

## 3. Treat per-user read state as its own subsystem

Rule: read cursors are not room metadata.

Discord gives read state a dedicated service because it changes at a different rate and has different ownership than room history. Keeping read state separate avoids locking room writes behind per-user cursor churn.

Source team: [Discord](https://discord.com/blog/why-discord-is-switching-from-go-to-rust)

```sql
-- Bad: one row tries to hold both room state and every user's read position.
UPDATE rooms SET last_read_seq = 42 WHERE id = $1;

-- Good: one monotonic cursor per (room, user).
UPDATE read_state
SET last_read_seq = $new
WHERE room_id = $room AND user_id = $user AND last_read_seq < $new;
```

## 4. Do not use one status field as workflow history

Rule: lifecycle progress needs durable steps, not just a label.

Temporal, Cadence, Conductor, Step Functions, and DBOS all exist because long-running progress is richer than `status = 'running'`. Once retries, timers, or approvals appear, the durable question becomes "what already happened?" not merely "what label are we in?"

Source teams: [Temporal](https://web.temporal.io/blog/workflow-introduction), [Cadence](https://cadenceworkflow.io/docs/concepts/workflows), [Conductor](https://docs.conductor-oss.org/devguide/concepts/workflows.html), [AWS Step Functions](https://aws.amazon.com/documentation-overview/step-functions/), [DBOS](https://www.dbos.dev/blog/dbos-transact-open-source-typescript-framework)

```ts
// Bad
workflow.status = 'processing';

// Good
workflow.history.push({ step: 'charged_card', at: now });
workflow.nextStep = 'send_receipt';
```

Two consequences from real failures. Each run or attempt gets its own durable identity; a calendar date is not one (a nightly job kept its state in a folder named for the date, so a manual run and the scheduled run on the same day collided and the scheduled one skipped). And a status anyone can see is derived from the obligation or event that it describes, never written by hand (hand-written "waiting" labels stayed on a roster long after the work they described had been paused). Check: two runs on one date, and pause or finish followed by a refresh of every place the status shows.

## 5. Add idempotency at the boundary as soon as retries can duplicate effects

Rule: ambiguous retries need operation identity.

Stripe treats the API boundary as part of the state machine: the retried operation has a stable identity. Without that, the system cannot distinguish "retry the same effect" from "perform a new effect."

Source team: [Stripe](https://stripe.com/blog/idempotency)

```ts
// Bad: duplicate retry can repeat the business effect.
await chargeCard(input);

// Good: correlate retries by boundary identity.
await idempotency.run(key, () => chargeCard(input));
```

## 6. If many code paths mutate one entity, make one coordinator instead of more rules

Rule: serialize ownership before inventing more guards.

Cloudflare, Orleans, and OTP make correctness come from one owner per entity. That is usually simpler than allowing many writers and trying to reconstruct order after the fact.

Source teams: [Cloudflare Durable Objects](https://developers.cloudflare.com/durable-objects/concepts/what-are-durable-objects/), [Orleans](https://learn.microsoft.com/en-us/dotnet/orleans/overview), [OTP](https://www.erlang.org/docs/26/design_principles/users_guide.html)

```ts
// Bad: any caller can mutate session state directly.
session.state = nextState;

// Good: all mutations pass through one keyed mailbox.
await mailbox.dispatch(sessionId, () => sessionActor.apply(command));
```

When two features legitimately write one record, one coordinator is not enough by itself: say which fields each writer owns, or how their versions merge, before either ships. Example: a background save and an editor save, each prepared on an older copy, committed on the same parent nearly simultaneously; the editor refuses versions that differ in more than text, so the record silently stopped saving and surfaced later as a version conflict. Check: interleave both writers' saves on one parent and assert both fields survive.

## 7. If the product promise is instant local feel, invest in sync early

Rule: sync is product infrastructure, not a later patch.

Linear's lesson is that local-first responsiveness becomes much harder if the system starts with polling and incidental cache invalidation. If the UX promise depends on immediate reads and optimistic writes, the sync layer deserves first-class design.

Source teams: [Linear](https://linear.app/now/scaling-the-linear-sync-engine), [localfirst.fm transcript](https://www.localfirst.fm/15/transcript)

```ts
// Bad: UI waits for round-trip before showing the change.
await api.updateIssue(input);
setState(await api.fetchIssue(id));

// Good: local write now, reconcile later.
store.applyOptimistic(mutation);
sync.enqueue(mutation);
```

## 8. Do not buy CRDT or OT complexity until the product truly needs concurrent shared editing

Rule: start from the authority model, not the trend.

Figma is the canonical warning against cargo-culting CRDTs. They studied the space, then deliberately used the smallest merge boundary their centralized model allowed.

Source team: [Figma](https://www.figma.com/blog/how-figmas-multiplayer-technology-works/)

```ts
// Bad: full document CRDT for a server-owned append-only feed.
const doc = automerge.change(roomDoc, ...);

// Good: property-level or stream-level coordination when one authority is enough.
roomStream.append(roomId, event);
```

## 9. Model the agent as a durable thread or workflow

Rule: agent progress is not "just another chat message."

Anthropic, OpenAI, and LangGraph all preserve more than plain text: they preserve session continuity, tool-call trace, checkpoints, and resume points. Once an agent can pause, fork, or call tools, its execution lineage deserves its own durable surface.

Source teams: [Anthropic](https://code.claude.com/docs/en/agent-sdk/sessions), [OpenAI](https://platform.openai.com/docs/guides/conversation-state), [LangGraph](https://docs.langchain.com/oss/python/langgraph/durable-execution)

```ts
// Bad: hide tool progress inside a normal chat message.
messages.push({ role: 'assistant', text: 'calling tool...' });

// Good: persist execution state separately.
thread.items.push({ kind: 'tool_call', tool: 'search', status: 'running' });
```

## 10. Use Hickey's test when the design feels muddy

Rule: ask which concerns are being braided together.

Rich Hickey's framing is useful because it catches the root mistake early: durable history, ephemeral awareness, workflow progress, and read state often look related in the UI while needing different invariants underneath. If one table or reducer is trying to do all of them, the design is probably mixing shapes.

Source thinker: [Rich Hickey, "Simple Made Easy"](https://www.infoq.com/presentations/Simple-Made-Easy/)

```ts
// Bad: one aggregate mixes unrelated concerns.
type RoomState = { messages: Message[]; typing: string[]; workflowStatus: string };

// Good: one join point, separate surfaces.
type Room = { stream: Stream; presence: Presence; workflow: AgentThread };
```

## 11. Preserve user-action meaning across requests and refreshes

Rule: sharing a transport primitive does not make user actions semantically interchangeable.

Before wiring an asynchronous action, distinguish creating an effect, observing an existing effect, and retrying an uncertain attempt. State the user's requested outcome, what is already known, and the allowed visible transitions, including pending and final wording. Derive feedback and available actions from that intent and evidence. A function name or an in-flight HTTP request is not evidence that the user's operation has started again.

Reuse request and receipt code below this boundary. An observational refresh must not replay a creation handler's UI transitions or erase confirmed progress without new authoritative evidence, even when the API uses an identical idempotent POST for both purposes. Idempotency prevents duplicate external effects; it does not prevent a shared callback from replaying misleading UI effects. Keep automatic receipt bookkeeping within the original user action unless another user decision is actually required. If completion was not yet confirmed, a failed observation leaves it unconfirmed; it does not establish that the underlying operation failed.

Verify the sequence the user sees as well as the eventual external result. Hold or lose a completion response after the effect commits, then exercise refresh or recovery. Assert the permitted user actions, wording and progress styling at each transition, the retained operation identity, and the external effect count. Do not add an extra test click merely because the implementation offers a button. Extend the existing state model and test fixture; this rule does not require a new workflow framework or test harness.

Example: in one web app, a shared send/check handler reset the UI to "Sending" during receipt checks, and its original test accepted a second send action. The corrected regression test requires automatic confirmation with no second button and preserves the original snapshot through later edits.

## 12. Make every read cost what changed, not how much history exists

Rule: a change notification is a delta instruction, not a refresh signal.

When a store emits change events that name the entity that changed, re-read that entity and merge it by identity. Read the whole collection only on first open, after a gap the events cannot account for (reconnect, cursor loss), or when an event names nothing the client holds. Event reads continue from their cursor and ask only for the kinds they use. A collection that grows forever and is refetched whole on every change is a defect even when each read is fast today, because its cost is proportional to history and its frequency to activity.

Before adding any read, state three numbers: what it downloads for the largest real entity, how often it repeats, and whether that grows with history. If the owner offers no way to read one record or continue from a cursor, ask for it; do not budget around it with longer debounce or bigger caches. Windowing (paging a list that the client scans to derive status) is not the fix: it makes records outside the window render as confidently wrong states. Omitting large bodies reduces cost but still grows.

Example: an unread check re-read the whole event log from the start on every change; continuing from the cursor and filtering by kind made the read proportional to what changed. A delivery-record list refetched whole on every change was fixed by reading only the record an event names, with a full read only on open or after a gap. A proposed paged list was rejected because it would have shown records outside the window as "never routed".

Repeat: sync between two machines. To tell a second machine that a new project was created, one design sent a nudge ("your project list may be stale") and had the receiver fetch the sender's whole project list and clone whatever it lacked. Two machines that deliberately hold different projects make every diff read as "missing". The event already knew the one fact that mattered: send it ("this project was added"). Full-state comparison (anti-entropy) is a repair tool for replicas that are meant to be identical, never the delivery mechanism for an event between parties that are not.

```ts
// Bad: any change refetches the whole, ever-growing list.
onEvent(e => { if (e.operationId) receipts = await api.listReceipts(sessionId); });

// Good: the event names what changed; read that one and merge by identity.
onEvent(e => {
  if (e.operationId && held) held = mergeById(held, [await api.getReceipt(e.operationId)]);
  else markStale(sessionId); // first open or unexplained gap: one full read
});
```

## 13. Decide an obligation only by explicit signals, never by reading prose

Rule: when one agent owes another an answer, the obligation closes only on a typed signal (a reply command with its disposition, a cancel, an execution failure), never on an interpretation of text.

A handed-off request is a durable workflow with one open obligation. Define its states, the explicit signal that moves each one and who produces it before writing any close path. A turn ending, a message that sounds final, or silence are not signals; they are prose, and reading them turns every agent's phrasing into a way to lose work. Every wait must end somewhere explicit: the worker is running, or waits on something the system tracks, or it is woken once and then the requester is told it stalled. Nothing waits in silence and nothing is guessed. When a transport answers, record receipt only for the exact event the other side confirms it holds; a plain success is not custody. When an old inference already closed an obligation, let the worker's later explicit answer supersede it as a new event rather than discarding it.

Example: a dispatch service closed requests by reading a finished turn's closing text as the answer or its silence as "unanswered". One request closed on "Final reply will follow"; later the worker's real final reply arrived, was discarded, and the worker was told it had been received. The first fix reordered the checks so a partial reply was seen first; the correction was to delete the prose reading entirely and design the whole protocol (states, reminder, stalled notice, per-event acknowledgement), including a typed command by which the worker declares its reply final.

The agent request and the person's task are different obligations. An explicit failed or cancelled agent reply settles that attempt; it does not settle the task that caused it. Keep the human task durably open, preserve the attempt and any effects, and route the remaining work again. A live question only the person can answer may suspend routing while it remains visible and answerable; the person's explicit withdrawal may close it. Do not turn an agent's provider error, hand-back, stalled turn, or closed decision question into an unowned terminal record. Check this with a marked task whose first attempt fails and whose checked replacement closes it.

Repeat: the first theory was that rerouting hand-backs would cover failures, but failed and cancelled attempts could still close work without a replacement, and lapsed questions could sit unowned. The fix left the task open through failed attempts and sent a durable recovery notice to the router; a marked failed attempt remained open and a checked replacement closed it.

```ts
// Bad: the turn ended, so treat its last words as the answer.
if (turn.status === 'done') settle(request, 'undetermined', turn.text);

// Good: only the worker's command closes it; an idle worker is reminded, then reported.
if (turn.status === 'done' && !workerWillWake(worker)) remindOnceThenReportStalled(request);
```

## 14. Separate the lifetime of what does the work from the lifetime of what coordinates it

Rule: the process you update often must not hold the lifetime of the process that must keep running.

The seven shapes say who owns ordering and mutation; they say nothing about which OS process holds
the pipes, and that is where one agent-hosting service broke. Every running agent was a child of the coordinator
service, so the service could not restart without killing it; updates waited for an idle moment
that never came, and the attempted fix held the user's own messages behind the longest-running
agent. The shape was right (one coordinator per session, a durable turn); the
lifetime was wrong. Anthropic's managed-agents team hit the same wall and moved the harness out of
the sandbox container ("the harness leaves the container… If the container died, the harness
caught the failure as a tool-call error"); Meta's XFaaS
keeps controllers out of the execution path so "controller downtime for tens of minutes" is
survivable.

The seven shapes do not cover this lifetime split. Ask, for every long-running thing: which process holds its pipes, which supervisor
kills it, what dies with it, and whether the thing you update most often is in that set. If it is,
give the work its own supervised lifetime and let the coordinator reattach by identity and replay
from a cursor.

```ts
// Bad: the coordinator spawns the worker; restarting the coordinator kills the work.
const child = spawn('claude', args, { stdio: ['pipe', 'pipe', 'pipe'] });

// Good: a separately supervised host owns the worker; the coordinator attaches by id.
const host = await startExecutionHost(executionId, manifest);   // its own unit, survives us
const stream = await host.attach({ afterSequence: log.cursor(executionId) });
```

## 15. Merge progress by identity before aggregating

Rule: when durable and provisional readings describe the same work, reconcile them for each entity first, then add the entities. For cumulative earned values, take the maximum known value per identity; never subtract an observed value from a total unless that total is guaranteed to include that very identity. Pending classification can keep an entity out of a list without erasing the fact it has earned progress. Check the whole transition from pending work through acknowledgement, sync and reload, not just the completed item.

Example: a countdown jumped back up every few seconds during dictation. Time was saved on an empty entry while the day's aggregate excluded empty entries until transcription supplied words, and the page subtracted the entry's observed credit from that aggregate, so each save erased the provisional gain. Earlier repairs left the formula intact and checked neither countdown-through-dictation nor reload. The repair gives each entry one credit identity, merges observed and retained seconds with a maximum, then sums the entries once.

## 16. A success signal proves only what it measures

Rule: before code settles, deletes or advances on a success signal, name the rung of the acceptance ladder that signal proves: recorded by us, admitted by the transport, saved durably by the receiver, or shown to the person. Settle only on the rung the obligation needs.

This is a frequent miss, because the shapes name the right owner but not the moment custody moves. Examples: a relay marked an idempotency key used when it accepted a message rather than when it was delivered; held messages were deleted before the receiving runtime accepted them, and after the repair a restart still erased context because "handed off" was recorded before the runtime saved the turn; a queue claim checked that an update changed exactly one row, but a trigger on that table made the database driver report more, so every claim committed and then returned "no claim" and dispatch stopped silently (fixed by reading the engine's own change counter).

Check: inject a failure just after each rung (receiver crash after admission, response lost after the receiver saved, restart before display) and assert no loss and no second effect. Where a database count or a transport status guards custody, test it on the installed runtime and schema, triggers included.

```ts
// Bad: the transport said ok, so the work is done.
if ((await relay.send(msg)).ok) await keys.markUsed(msg.key);

// Good: settle on the rung the obligation needs, named by the receiver.
const receipt = await relay.send(msg);            // admitted
await keys.markPending(msg.key, receipt.id);
onDelivered(receipt.id, () => keys.markUsed(msg.key)); // saved by the receiver
```

## 17. An older installed reader will meet every new record

Rule: a new record kind, field or message format ships only with a check that a still-open older page, service worker or peer can read it or pass it through, keep its own pending writes, and report a typed incompatibility instead of stopping.

Installed clients do not update together. Examples: a release wrote one block with a new property, and an already-open older page could not decode it, parked replication and held its local changes; a push queue began requiring an acknowledgement that older installed service workers never sent, so users saw stale notifications. The sync-engine shape names replicas but not that the oldest live reader is part of the contract.

Check: run the previous release's decoder (or worker) against records the new release writes, before deploying.
