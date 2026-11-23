# Select signals from operational questions

Start with the user-visible outcome and expected workload. A nearly idle personal
service needs availability, freshness and backlog checks; a high-volume API may
also need error-budget and latency-percentile alerts. Instrument normal operation
and impending degradation as well as incidents.

| Workload | Useful signals | Health question |
|---|---|---|
| HTTP/API | Request attempts, final status/outcome, duration histogram, in-flight gauge, dependency failures | Can callers complete useful requests within the expected time? |
| Local-first app | Storage open/commit/refusal, reconciliation state, rendered outcome, retained/dropped diagnostics | Are edits preserved, and does the UI accurately represent authoritative state? |
| Queue/detached work | Durable depth and oldest pending age, attempts versus unique terminal effects, processing duration, parked work | Is accepted work progressing, including when no worker is running? |
| Scheduled job | Expected schedule, last completed success, native invocation result, duration, missed/stale completion | Did the job succeed when it should have, including failure to start? |
| External dependency | Attempts, latency, timeout/rejection class, circuit/availability state where present | Is the dependency delaying or preventing the application outcome? |
| Host/runtime | Disk bytes and inodes, memory/CPU pressure, relevant pool/event-loop saturation, restarts | Will resource exhaustion disrupt the app before an operator can react? |
| Telemetry pipeline | Producer/collector freshness, scrape failures, dropped/rejected samples, backlog or disk bound | Is a quiet view healthy or blind? |
| Product-specific outcome | A small aggregate measure tied to a real concern: successful backup freshness, failed saves, reconciliation lag | Is the service doing the job users depend on? |

Logs explain specific events; metrics summarize behavior over time; traces show
causal work and latency across boundaries. Use each when it answers a question.
Do not add distributed tracing to a local utility solely to fill three columns.

## Logs and trace context

Prefer structured events with a stable event name, severity, UTC timestamp,
service/environment/build and a small typed field set. Severity reflects operator
attention: expected refusals are not all errors; repeated healthy polls are not
necessarily useful logs. Record a safe failure code and stage rather than a raw
exception string. Retain local debug detail only under an explicit privacy policy.

Separate causality from transport and effects:

- Logical action/operation ID: survives its attempts; avoid counter reuse across
  reloads when old history remains. Local/offline work can have no HTTP request.
- Attempt ID: generated before starting transport, retained on failures without a
  response. A retry creates a new attempt ID and preserves the logical effect key.
- Service request ID: owned by that server; downstream IDs are separate fields.
- Trace/span context: follow supported W3C/OpenTelemetry propagation; spans describe
  their actual parent or links, including fan-out/detached jobs. Never attach work
  to the most recent action merely because it is convenient.
- Effect/receipt ID: joins to the authoritative commit/delivery result. A successful
  HTTP response does not automatically prove the end user's operation completed.

Receive, validation/rejection, completion/disconnect, durable acceptance, terminal
work outcome and visible acknowledgment are meaningful observation boundaries.
Record the ones required by the app's questions. Pair a stage with an explicit
terminal outcome; a failure at `start` must not look like another start event.
Cancellation of foreground observation does not cancel already accepted work.

## Metric semantics

For each metric establish its source, unit, type, dimensions and reset behavior:

- Counters count occurrences, reset with their owner, and use reset-aware rates.
  HTTP attempts count retries; unique effects count authoritative transitions.
- Histograms use seconds/bytes or another explicit base unit and buckets meaningful
  for the workload. Count bucket expansion in the series budget.
- Gauges represent current authoritative state. Derive durable backlog and oldest
  age from the actual queue; an in-memory attempt counter cannot reconstruct them.
- Duration uses a monotonic clock; timestamps use UTC. State explicitly whether
  freshness is observation time, last success time, or source completion time.
- Enumerate bounded labels such as route template, method class, outcome, queue or
  configured job. Normalize unknown values to a bounded fallback and test it.
  Budget the label cross-product and histogram series, not just metric names.
- Emit a meaningful zero for a known empty queue/zero traffic. Missing source state
  must remain unknown, not become zero backlog or a fresh successful job.

Automatic SDK instrumentation is a starting point. Inspect resource attributes,
HTTP paths/query strings, database statements, exception text and AI spans before
export. Standard conventions improve interoperability but do not prove privacy.

Sources: [OpenTelemetry signals](https://opentelemetry.io/docs/concepts/signals/),
[W3C Trace Context](https://www.w3.org/TR/trace-context/),
[Prometheus instrumentation](https://prometheus.io/docs/practices/instrumentation/),
[metric naming](https://prometheus.io/docs/practices/naming/).
