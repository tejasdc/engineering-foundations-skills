# Sources, and what each is good for

Order of trust for a behaviour claim: a measured probe on the installed version; the platform's
pinned source at a commit; the installed SDK's type declarations; the canonical documentation
page, fetched and quoted; a blog or forum post (a hypothesis). Say which one you have.

## The project's own record (first, always)

- Retrospectives, incident notes and design plans in the project, and any lessons log.
- Decision records, with their rationale.
- The code and supervisor configuration: `systemctl cat <unit>`, launchd plists, the spawn
  call, the retry policy as implemented.

## Reading worth knowing

- Anthropic, "Scaling Managed Agents: Decoupling the brain from the hands": split session,
  harness and sandbox; `wake(sessionId)` and resume from the session log's last event. Lifetime
  separation.
- Meta's XFaaS: control plane out of the execution path; "controller downtime for tens of
  minutes". Lifetime separation; a test to copy.
- Alvaro Duran, "Why Payments Engineers Should Avoid State Machines": pull a log by cursor; "a
  state machine cannot reconstruct the past". Logs and cursors.
- Eric Allam, "Two Roads to Durable Agents: Replay vs. Snapshot" (AI Engineer 2025 talk): replay
  logs need determinism and are hard to version across deploys. Decide replay vs snapshot
  explicitly.
- Mason Egger, "Events are the Wrong Abstraction for Your AI Agents" (AI Engineer 2025 talk):
  durable execution versus ad hoc pub/sub.
- Vinoth Govindarajan, "Your Agent Didn't Fail. Your Harness Did." (AI Engineer 2026 talk): record
  what woke it, which authority, what executed (attempt, idempotency key), what evidence survived.
- clig.dev, Command Line Interface Guidelines: recoverable and crash-only programs.
- Alan Kay via Scott Werner, "As Complexity Grows, Architecture Dominates Material", and Stephen
  Merity's notes on "Rethinking Design, Risk and Software": architecture versus more bricks;
  construction needs its own infrastructure.
- Richard Cook, "How Complex Systems Fail": no boundary is sufficient alone; observe.
- Foote and Yoder, "Big Ball of Mud": piecemeal growth is the default; keeping it working while
  refactoring is the alternative to a rewrite.
- Kleppmann, Designing Data-Intensive Applications (read the chapters): logs, fencing tokens,
  idempotency, exactly-once, consensus.
- Kyle Jaejun Lee, "I Run a Fleet of AI Agents Across Three Machines" (AI Engineer talk): "your
  one point of control can't be a thing that falls asleep"; machine-local versus shared state.

## Established references (as principles at this scale)

- AWS Well-Architected, current best-practice ids: REL05-BP01 graceful degradation; REL04-BP02
  loosely coupled dependencies; REL04-BP04 make mutating operations idempotent; REL05-BP03 control
  and limit retry calls; REL10-BP03 bulkheads (principle only: no cells at this scale); REL08-BP04 immutable
  infrastructure (does not apply without a fleet); OPS06-BP04 automate testing and rollback; OE design
  principles "implement observability for actionable insights", "learn from all operational
  events". `docs.aws.amazon.com/wellarchitected/latest/...`. Quote the current title; titles change.
- Erlang supervision principles (`erlang.org/doc/system/sup_princ.html`): restart strategies per
  child; never an unconditional restart for an effectful worker. ("Let it crash" is elsewhere.)
- Temporal, workflow message passing: signals as asynchronous writes to a sleeping logical
  execution; no deterministic replay of LLM or shell execution here.
- FIPA request: accepted and finished are separate facts. Transactional outbox; idempotent
  receiver; store-and-forward; end-to-end argument (Saltzer, Reed, Clark).
- systemd `systemd.kill` and `systemd-run` manuals (freedesktop pages sit behind a bot wall;
  the Debian manpage mirror carries the same text): `KillMode=control-group` versus `process`
  (not recommended: processes escape the manager); a transient service is PID-1-parented, a
  scope stays the caller's child. launchd `launchd.plist(5)`: `AbandonProcessGroup`, `KeepAlive`.
- Provider documentation: code.claude.com (agent view, scheduled tasks, sub-agents, hooks, Agent
  SDK), OpenAI Codex documentation (app server, hooks, subagents), `github.com/openai/codex` at a
  pinned commit. Stream event names not on a docs page are cited to the installed SDK types.

## Skills this one routes to

[stateful-shapes](../../stateful-shapes/SKILL.md) (state inside one system),
[protocol-design](../../protocol-design/SKILL.md) (exchange between parties),
[architecture-lessons](../../architecture-lessons/SKILL.md) (one home per fact),
[application-observability](../../application-observability/SKILL.md) (signals and alarms),
[review-gates](../../review-gates/SKILL.md) (review level and loop).
