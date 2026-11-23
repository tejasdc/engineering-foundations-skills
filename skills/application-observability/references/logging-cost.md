# Logging that never slows the application

Logs and metrics are a workload. They spend the writer's processor and memory, the sink's disk,
and the history every later investigation reads. Each rule below carries the source or
measurement that earned it. The requirement: adding the logging and metrics itself must not make
the app slow; it should all be background work on separate threads or asynchronous operations.

## What goes wrong

Failure modes seen when several apps write to one shared log sink such as a system journal:

- **Blocked writers.** When the log reader pauses, a runtime's console writes can block or queue.
  A probe that paused the reader for 3 s showed Bun's `console.log` freezing its event loop for
  2.9 s, while Node's `console.log` kept going but queued about 510 MB of memory in 6 s.
- **Client amplification.** A browser page that logs each request and decode line, and uploads
  each as one server line, can multiply into tens of thousands of server lines in minutes and
  overflow its own upload queue. Client diagnostics re-logged one-for-one became most of one
  app's volume.
- **Disk full from a copy loop.** A syslog daemon copying the whole journal into a second file
  doubled disk writes. When the disk filled it logged each failed write, the journal handed those
  lines back, and an error loop followed.
- **Retention shorter than the follow-up.** At high volume a size-capped journal can hold only a
  few days. An agent that adds a log and returns "in a couple of days" may find almost nothing.
- **Shared disk contention.** The sink waits on the same disk that other heavy processes
  saturate, so every app that writes through it waits too.

Earlier lessons in the same family are in [SKILL.md](../SKILL.md): a conditional log looked like
full coverage, and a job logged its own failure into a place nothing read.

## Rules

1. **The request path never waits for the log sink.** Write each record into a bounded in-process
   buffer and return. A separate flusher drains it to the sink: a worker thread, or a writer that
   writes only while the descriptor is writable. When the buffer is full, drop the record and
   increment a counter that is exported as a metric. *Why:* systemd connects stdout to a journald
   stream socket. When journald stalls, Bun's `console.log` blocks the event loop (2.9 s in the
   probe), and Node's queues without bound (510 MB in 6 s). [Node documents](https://nodejs.org/api/process.html#a-note-on-process-io)
   that writes to pipes and sockets are asynchronous on POSIX, which means they queue rather than
   block. Bun's blocking was measured, not documented. OpenTelemetry's
   [batching processor](https://opentelemetry.io/docs/specs/otel/logs/sdk/) is the reference shape:
   a 2,048-record queue, exports every second, and drops when full. *Enforce:* a native test pauses
   the reader and asserts the event-loop gap stays under 100 ms and memory stays bounded.
2. **Formatting is on the hot path too.** Build records from a few small fields. Never serialize
   whole objects, payloads or arrays per line. Defer any expensive formatting to the flusher.
   *Why:* the bounded buffer protects against a slow sink, not against the processor time spent
   formatting each line before it is buffered.
3. **A record of every success is a metric, not a log.** Count successes and time them in
   histograms that are already scraped. Log failures, slow operations and a fixed sample of
   successes. *Why:* per-request started/completed lines and per-read phase lines can dominate
   an app's volume while carrying no more information than a histogram.
4. **Never mirror one log stream into another line for line.** Aggregate at the source and ship
   counts, failures and samples. *Why:* re-logging browser lines on the server can dominate a journal, and
   copying the journal into a second file doubles disk writes and can create an error loop.
5. **Clients sample and rate-limit before they upload.** Use a bounded queue, a fixed rate ceiling
   per page, and a drop counter that reaches the server. Never write to synchronous storage, such as
   `sessionStorage` on the main thread, for every event. *Why:* the amplifying page above. A
   client cannot know whether the server is struggling, so the ceiling must hold without that
   knowledge.
6. **A logger never reports its own failure to the same sink.** Count a sink failure in a metric,
   or write it once per interval to another destination. *Why:* the disk-full error loop above.
7. **Levels mean something.** In production, `info` records state changes and outcomes. `debug`
   is off by default. A temporary investigation log says when it should be removed, and the
   come-back step below removes it. *Why:* investigation logs that are never removed become the
   permanent volume of rule 3.
8. **Budgets come from what investigations need, not from what fits.** Set retention to at least
   the longest planned come-back plus a margin. Divide the sink's capacity by that retention to
   get the whole host's write budget, and give each writer a share. *Why:* a capacity that looks
   large can work out to only a few KiB/s for the whole host over a week, which one chatty app
   can exceed alone. Raising the cap, deleting history or muting the alert hides the cost; it does not fix it.
9. **Bounded labels; never private content.** The cardinality and privacy invariants in SKILL.md
   apply to every new field. Prometheus [warns](https://prometheus.io/docs/practices/naming/#labels)
   that each label combination is a new time series. Never log note text, messages, prompts,
   credentials, one-time codes, or URLs carrying a token. *Why:* logs leave the process, persist
   and are read by agents. A leak cannot be recalled.
10. **Watch the logging system from outside every writer.** Measure lines and bytes per writer and
    per event, lines the sink discarded, receipt lag (when the sink received a line minus when the
    writer stamped it), the time a test write takes to become readable, disk held by log stores,
    and how far back history reaches. Each budget breach goes to whoever operates the host (or an
    on-call agent), not to the end user. *Why:* a writer whose logging
    blocks cannot report that itself. Receipt lag sees it from outside. Keep these alerts, for
    example "logging over budget" and "logging slows requests", in the host's observability docs.

## When evidence is missing: add the signal, then come back

When an investigation cannot conclude because a log or metric is missing, adding that signal is
only half the work. Also schedule a follow-up (a scheduled agent run, a timer, a calendar
reminder) that says "now that the logs exist, check them and actually fix the issues". Adding
logs and resting is not finishing.

1. Add the signal under the rules above, deploy it, and confirm one real record arrives.
2. In the same change, schedule the follow-up using whatever scheduling facility your
   environment offers. Nothing expensive should run until it fires.
   - Prefer triggering on the evidence itself: a short script that exits once enough matching
     records exist, or once the condition you are waiting for has happened, plus a deadline that
     catches the case where it never does. Keep that check short-lived: a restart of the
     supervising service can end a long waiting command and fire it early. For "come back at a
     time", use a deadline-only trigger, which survives restarts.
   - Watch a file when the evidence lands in a file. Start a fresh session only when a fresh
     session should do the follow-up.
   - The deadline falls after the deployment has had real traffic. It also falls well inside the
     sink's retention (export the retained-history length as a metric and check it): a follow-up
     that lands after the history has expired finds nothing.
3. Name the follow-up in the report and in the commit, with its time and the question it answers.
4. When it fires, read the evidence, then fix the problem or say exactly what is still unknown.
   Remove or downgrade the temporary log in the same change.

What enforces this: only this procedure, together with the scheduler's own record of what is
registered. A check that refuses to close a change which adds an investigation log without a
scheduled follow-up would be stronger; where none exists, this section is the fallback.
