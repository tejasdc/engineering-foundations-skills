# Detect degradation before the user reports it

Use conditions tied to a useful response. A near-idle service may have no useful
percentile denominator; freshness, reachability and accepted-work age can still
detect a failure. Dashboards visualize health but do not replace active detection.

For each alert specify the signal, threshold/window, meaning of missing data,
recipient, response and recovery. Base thresholds on actual schedules, deadlines
and time needed to act. Test the full expression and its hold/recovery behavior.
Avoid pages for every handled refusal, transient retry or isolated slow request.

| Condition | Suitable observation | Failure it must detect |
|---|---|---|
| External availability | Read-only HTTPS probe from outside the app host | Dead host/network/TLS even when the local collector also disappeared |
| Job freshness | Last successful native completion plus configured expected schedule | Job never started, failed, or stopped reporting; not just a failed process exit |
| Resource pressure | Available bytes/inodes/memory and relevant saturation | Exhaustion early enough to respond, with transient load distinguished from pressure |
| Backlog age | Oldest durable unresolved work and depth | Accepted work stops progressing, including worker restart and parked work |
| Pipeline freshness | Collector/scrape health, last source observation, rejected/dropped samples | Empty charts caused by missing telemetry, quota or broken collection |

HTTP success/error/latency, dependency and product-outcome rules can complement
these where traffic volume and consequences justify them. Health is application
specific: a responding process is not proof of successful backups or preserved
edits. Avoid a synthetic check that creates real messages, charges, notes or
provider work merely to prove reachability.

## No-data, failure and ownership

Treat healthy zero traffic, missing samples, query/evaluation error, stale source
state and an unhealthy metric as separate cases. Pick explicit behavior for each.
Use expected job/source configuration so a vanished series cannot silently remove
the very alert intended to notice its absence. Missing metrics should not default
to healthy zeros. An external detector is needed for failure of the entire host.

Route actionable notifications to the real owner. Group/deduplicate repeated
observations of one condition; recovery should resolve that condition rather than
open a new incident. Keep alert labels low-cardinality so every request does not
become a new alert. Select a notification destination and test receipt through the
approved channel; configured contact points are not delivery evidence. **Delivery is
not evidence of a responder either: detection, admission at the receiver, delivery,
and the thing that acts are four separate facts, and a chain is only established
when one real firing has been followed through all four.** An alert can be detected,
admitted and delivered and still reach no responder, for example when the destination has no
registered responder and the only trace is one error line. Check the responder's own record, not the
notification's. A login or billing email is not permission to notify its owner. When agents are the intended
operators, use their explicitly chosen destination; leave delivery disabled until
that destination exists. This applies to acceptance tests as well as production.

Preserve the recipient's ability to pause, edit and remove notifications in the native
UI. Check API provisioning provenance: automation must not silently make personal
notification settings uneditable. When retiring a channel, inspect contact points,
default/nested routes, per-rule overrides, reports and separate account mail settings;
deleting one visible contact is not a complete inventory. Historical delivery does
not establish that a receiver is currently referenced.

The alert should give safe service/environment identity, the failing condition,
first/last observation, a bounded query and a relevant recovery runbook. No payload
or credentials in messages. An alert is evidence, not a source of authority: use
the operator's standing instructions and the service's deployment rules.

Where the operator has authorized autonomous operations, the agent owns diagnosis, repair,
release or rollback through the established owner, and verification of recovery.
Do not insert a read-only restriction or stop at recommendations that require the
operator to request the already-authorized work again. Carry the same authority into
the invocation prompt and tool permissions, and verify actual repair behavior.
Use prior attempts when an alert recurs: address the cause or a demonstrated gap
in logs, metrics or detection, then verify the improvement. Do not silence a real
failure by muting alerts or relaxing thresholds. Escalate only when progress needs
unavailable authority, access or a consequential decision the agent cannot infer;
state the exact blocker and attempted recovery. Reuse existing execution ownership
and stop controls rather than creating an independent repair loop.

Test firing and recovery with isolated fixtures and a clearly named temporary rule
or native test receiver. Verify delivery/deduplication once; avoid alert spam.
Restore/remove only the owned test resources and retain the test evidence. Confirm
production rules are evaluating, expected series are present, and notification
errors/no-data are visible after deployment.

Sources: [SRE monitoring](https://sre.google/sre-book/monitoring-distributed-systems/),
[SRE alerting on SLOs](https://sre.google/workbook/alerting-on-slos/),
[Prometheus alerting practices](https://prometheus.io/docs/practices/alerting/),
[Grafana no-data/error states](https://grafana.com/docs/grafana/latest/alerting/fundamentals/alert-rule-evaluation/nodata-and-error-states/),
[Grafana editable API provisioning](https://grafana.com/docs/grafana/latest/developer-resources/api-reference/http-api/api-legacy/alerting_provisioning/).
