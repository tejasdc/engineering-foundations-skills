# Shape 7: Durable conversation / agent thread

## Definition

A durable agent thread is a conversational or task execution that carries more than chat text. It accumulates tool calls, checkpoints, human interventions, forks, and resumable progress across turns. The key design move is to treat the thread or session as the unit of continuity rather than the latest message or prompt string.

## Where it occurs

- Anthropic Claude Agent SDK sessions
- OpenAI conversation or response chains
- LangGraph state graphs with checkpointers
- Modern tool-using agent runtimes

## Key invariant

The unit of continuity is the thread or execution, not just the latest message.

## Illustration: a reply destination read narrower than the displayed thread

In a multi-agent thread, a user replied to an older agent's post but the composer offered only the coordinator, because the recipient query read posts from the current coordinator session while the visible thread read all of them. For a replyable conversation, the read that decides recipients must cover the same retained message lineage as the read that displays the conversation. Keep the exact post author eligible after its original request closes, and follow an accepted session handoff. Check the visible composer choice, outgoing target, accepted receiver and answer location separately.

## Illustration: a held message declared delivered before it was saved

A gateway moved untagged group messages into a tagged turn and marked them handed off before the agent runtime durably saved that turn; on restart it deleted those rows, so a crash between handoff and save could erase context. For any thread that moves context between stores, identify the exact durable acceptance point. A callback that starts a run, or a lease marked handed off, is not proof of acceptance. Review the producer, the destination's persistence boundary, restart settlement and the user-visible reply boundary together. When the runtime offers no acceptance receipt, state the at-least-once or at-most-once tradeoff explicitly and choose against silent loss of human context. An archive is reference custody, not proof that the next model turn has loaded it. The general form is [guideline 16](../guidelines.md#16-a-success-signal-proves-only-what-it-measures).

## Illustration: a handoff kept requests but lost visible ownership

When a coordinator agent hands its open requests to a successor, transfer custody and presentation together: retain the original thread as the return destination, accept the exact open-request inventory once, show the successor's identity (provider, machine, session), and publish one current plan with each workstream marked live, in progress or unverified. A successor must correct a false visible tracker claim, not repeat it. A handoff receipt proves request custody, not that the user can see who is coordinating.

## Mental model, in teams' own words

- Anthropic: "How sessions persist agent conversation history, and when to use continue, resume, and fork." Source: https://code.claude.com/docs/en/agent-sdk/sessions
- OpenAI uses conversation objects or `previous_response_id`, and conversations store messages, tool calls, and tool outputs as items. Source: https://platform.openai.com/docs/guides/conversation-state and https://platform.openai.com/docs/api-reference/conversations?api-mode=responses
- LangGraph: "If you are using LangGraph with a checkpointer, you already have durable execution enabled." Source: https://docs.langchain.com/oss/python/langgraph/durable-execution
- LangGraph interrupts "pause graph execution at specific points and wait for external input." Source: https://docs.langchain.com/oss/python/langgraph/human-in-the-loop

## When to use this shape

- An agent can call tools, wait, resume, or fork.
- Human approval or intervention must pause and later continue execution.
- The runtime needs to remember more than plain text history.
- Tool outcomes and checkpoints need durable auditability.

## When NOT to use this shape

- The feature is a plain append-only conversation with no resumable execution.
- The hard part is ordering one room's messages, not preserving agent lineage.
- The right abstraction is a durable workflow with minimal conversational context.

## Compatible libraries / patterns

| Library or pattern | Adopt when... |
| --- | --- |
| Session or conversation object | The main need is durable conversational continuity and tool trace. |
| LangGraph | The agent needs explicit graph state, checkpoints, and interrupts. |
| Temporal / DBOS | The agent is embedded inside a heavier durable workflow. |
| Append-only thread plus checkpoint rows | A lightweight runtime needs explicit resume points without full framework adoption. |
| Plain chat transcript only | Overkill warning in reverse: this breaks once tools, forks, or waits are first-class. |

## TypeScript/Node applicable example

```ts
type ThreadItem =
  | { kind: 'user'; text: string }
  | { kind: 'tool_call'; tool: string; status: 'running' | 'done' }
  | { kind: 'checkpoint'; node: string; state: Record<string, unknown> }
  | { kind: 'interrupt'; reason: string };

type AgentThread = { id: string; cursor: number; items: Array<ThreadItem & { cursor: number }> };

export function appendItem(thread: AgentThread, item: ThreadItem): AgentThread {
  const cursor = thread.cursor + 1;
  return {
    ...thread,
    cursor,
    items: [...thread.items, { ...item, cursor }],
  };
}
```

## References

- https://code.claude.com/docs/en/agent-sdk/sessions
- https://docs.anthropic.com/en/docs/claude-code/sdk
- https://platform.openai.com/docs/guides/conversation-state
- https://platform.openai.com/docs/api-reference/conversations?api-mode=responses
- https://docs.langchain.com/oss/python/langgraph/durable-execution
- https://docs.langchain.com/oss/python/langgraph/human-in-the-loop
