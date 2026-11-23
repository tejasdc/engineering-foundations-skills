# Enforce behavior through existing native checks

Select tests by the changed behavior and repository release policy. The implementer
owns the evidence; this skill does not require a separate logging agent, mandatory
review or a new CI platform. If review is requested, provide the original
requirements/amendments separately from the design so author-added exclusions can
be challenged before implementation details.

Use the application's real shared boundaries and synthetic data. Keep regression
oracles independent of the implementation: known operations and outcomes determine
expected events and metric deltas. A test that repeats the event constructor does
not establish useful evidence.

| Contract | Meaningful native check |
|---|---|
| Shared adoption | Existing type/lint/architecture checks reject named raw logging/transport bypasses; new routes/events use the shared contracts |
| Logs and causality | Observe emitted success/refusal, transport-without-response, decode failure, interrupted/detached work and recovery; preserve attempt/effect distinctions |
| Local diagnosis | Storage failure still leaves usable local evidence; retained IDs remain unique after reload; bounded history reports evictions |
| Visible outcome | Assert the rendered state after authoritative transitions, including delayed completion and recovery; a recorder call alone does not prove the badge changed |
| Metrics | Fixture outcomes match counter deltas, duration buckets and current gauges; reset, no-traffic, missing source and label expansion behave as defined |
| Monitoring | Native rule tests cover healthy, degraded, missing/stale, firing and recovery; an isolated notification proves receiver delivery/deduplication |
| Privacy | Inspect actual exported/forwarded records with private sentinels placed in raw paths, payloads, headers and error strings; only approved fields leave |
| Non-interference | Disabled, offline, rejecting, full and restarted sinks cannot change successful product operations; loss/freshness and resource bounds remain visible |
| Agent access | Native query recipes answer an operation, a health and an incomplete-evidence question with independently known answers |

For each important oracle, include the smallest corresponding bad fixture: a
missing terminal outcome, reused attempt ID, unbounded label, stale producer or
private field. The check must reject it. Types/static checks can cover known
declarations and syntax; do not describe them as proof of every dynamic/library
call. Native runtime fixtures cover exported values and actual behavior.

## Hosted and collector acceptance

Validate the actual collector/rule configuration with upstream tools, then exercise
the same collector using isolated sources. Capture the outgoing data; inspect
fields and bounded buffers/retry behavior. Keep failed evidence rather than
weakening assertions or changing its labels.

For backend privacy checks, first prove the expected safe synthetic record and
metric reached the selected store under the correct source/build. Then query for
the private sentinel and inspect outgoing fields. An empty backend with no safe
record proves nothing about privacy. Scope synthetic probes so they cannot replay
real notes/messages, trigger real provider work or modify application data.

Confirm ingest acknowledgment, query availability, rule evaluation and delivered
notification separately. A successful API response or deployed unit is not all
four. Check the selected account's actual quota/retention and enabled rule/contact
point settings. Verify export/config recovery; verify a backup restore when the
new backend owns self-hosted state.

Bind source/config and test evidence through the repository's existing release
mechanism. Report implemented, verified and deployed separately, with exact
remaining missing access or live checks. Account provisioning can wait for the
credential owner while independent code/tests continue; it does not justify
calling an unconnected stack deployed or abandoning authorized implementation.

Sources: [Prometheus rule testing](https://prometheus.io/docs/prometheus/latest/configuration/unit_testing_rules/),
[OpenTelemetry sensitive data](https://opentelemetry.io/docs/security/handling-sensitive-data/),
[Why existing logs fail diagnosis (OSDI 2012)](https://www.usenix.org/conference/osdi12/technical-sessions/presentation/yuan).

Design note: repeated failures across applications tend to show incomplete operational signals,
missing proactive monitoring and unusable correlation, and reviews that inherit a narrow author
brief miss the requester's broader request. The reusable remedy is native behavior/evidence
checks plus independently stated requirements, not more prose or a permanent extra reviewer for
every edit.
