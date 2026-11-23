# Choose the smallest adequate collection and query system

Inventory live destinations, entitlements, access and retention first. Do not
confuse a package/configuration file with a working account or ingested data.
Prefer existing infrastructure when it can answer the operational questions.

| Choice | When it earns its place | Work we still own |
|---|---|---|
| Local structured files/journald with exact queries | Local/offline tools, small volumes, short investigations | Retention, rotation, schema, coverage, collection access; the host may be unavailable during its own outage |
| Platform-native logs/metrics | Most work runs on one platform and native query/health features fit | Instrumentation, permissions, privacy, sampling, alert semantics and any off-platform collection |
| Managed observability service | Mixed hosts/jobs need centralized search, metric storage, ongoing rule evaluation and external probes | Collector, access, quotas, signal quality, rules and provider outages; hosted does not mean maintenance-free |
| Self-hosted established backend | Data location, offline access or actual cost/retention needs justify operating it | Patching, TLS/auth, storage, backup/restore, capacity, query protection and independent monitoring |

Recommend a choice using workload, operator effort, data sensitivity, required
queries/monitoring, existing access and reversible cost. If uncertainty does not
change that choice, implement and observe it. Compare alternatives on a concrete
problem; do not require a general agent-usability competition before building.

Managed examples include Grafana Cloud and Axiom; platform-native examples include
Cloudflare's Workers observability; self-hosted examples include OpenObserve and
Grafana/Prometheus/Loki. These are options, not a universal vendor mandate. Check
current official APIs, licensing, limits and prices when selecting or installing.
Distinguish a source-available license from open source.

## Agent usability and files

An API/CLI with exact time/field selection, aggregations, schema discovery and
bounded/paginated output is a real investigator interface. A dashboard-only
integration is insufficient when agents are the primary users. Native query
languages can be acceptable; provide tested recipes and preserve query errors.

Files with JSON/SQL/exact ID search can answer many of the same questions. A
backend earns its cost by providing maintained remote collection, indexing,
retention, metrics and continuous monitoring. Semantic search can discover similar
symptoms; exact queries and authoritative receipts establish counts and outcomes.
Building our own indexing/alerting service also creates infrastructure to operate.

Keep signal meaning independent from the backend and collection in the host's
deployment channel. Portable collection reduces lock-in; changing stores can still
require query/alert rewrites and separate historical retention/export work.

## Collection, cost and recovery

Select reviewed sources rather than forwarding the host's entire journal. Filter
private fields before they cross the export boundary. Avoid duplicate forwarding
through multiple collectors. Keep ingestion, investigation and configuration
access usable by their actual owners, outside Git/argv/logs. Respect existing grants
and explicit owner authorization; automatically derive service credentials from
authorized administration access rather than repeatedly burdening the owner.
Read-only credentials are a useful default for investigation tools, while the
operator retains the separately authorized management path.

Estimate volume from observed events/bytes and series × scrape frequency; include
histogram/label expansion, synthetic-check locations/frequency, retention, egress
and query costs. Confirm the selected account's actual plan. Do not enable a paid
plan or overage without authorization. Free quotas can reject/drop data; monitor
usage and rejection instead of treating the free tier as unlimited or durable.

Set a retention window long enough for the expected investigation delay. Bound
local disk, spool, buffer and retry use and define the loss policy. Keep the app
available when the sink is offline, over quota, full or restarted. Expose those
conditions without recursively logging every failure to the same broken sink.

Store collector, rule, query and provisioning source with its owner. Use native
unit/host deployment and managed service APIs. Test config recovery/rollback and,
for a self-hosted stateful backend, restoration from a real backup. The only copy
of the evidence and monitor should not share the monitored failure boundary.

Sources: [Cloudflare Workers observability](https://developers.cloudflare.com/workers/observability/),
[Grafana Alloy](https://grafana.com/docs/alloy/latest/),
[Grafana pricing](https://grafana.com/pricing/),
[OpenObserve](https://github.com/openobserve/openobserve),
[Prometheus storage](https://prometheus.io/docs/prometheus/latest/storage/).
