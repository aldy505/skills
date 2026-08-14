# RED, USE, and Symptom-based Alerting

## RED metrics — request-driven services

For every service handling requests, track:

| Metric | What it captures | How to compute |
|--------|------------------|----------------|
| **R**ate | Requests per second | Counter / time window |
| **E**rrors | Error count or rate | Counter of `status_code >= 500` or `span.status = ERROR` |
| **D**uration | Latency (p50, p95, p99) | Histogram of request duration |

Use RED with an **SLO** (Service Level Objective) to decide what to alert on: alert when error budget is burning too fast, not when a single error occurs.

## USE metrics — resources (compute, network, storage)

For every resource:

| Metric | What it captures |
|--------|------------------|
| **U**tilization | % busy (CPU, memory, disk, network) |
| **S**aturation | Queued work / waiting (queue depth, thread pool size, connection pool) |
| **E**rrors | Resource-level errors (disk errors, NIC errors, socket errors) |

## Symptom-based alerting rules

Good alerts respond to user-visible symptoms, not internal causes.

**Threshold + duration pattern:** alert when `metric >= threshold` for `duration` seconds. Single-point spikes are usually noise.

**Examples:**

| Alert | Rule | Rationale |
|-------|------|-----------|
| High error rate | `http.errors / http.requests > 0.01` for 5 min | >1% error rate sustained |
| Latency p95 | `http.duration.p95 > 2s` for 5 min | p95 users seeing slow responses |
| SLO burn (fast) | `burn_rate_1h > 14.4` for 5 min | 14.4x burn rate = all budget consumed in 1h |
| SLO burn (slow) | `burn_rate_6h > 1.0` for 30 min | Slow sustained burn; alert before budget gone |
| Queue saturation | `queue.depth > 1000` for 10 min | Backing up; latency will follow |
| CPU saturation | `cpu.utilization > 0.85` for 15 min | Sustained pressure; may be heading for throttle |

## Two-tier severities

- **Page (critical):** user-visible, revenue-impacting, SLO fast-burn, data loss. Requires immediate human response.
- **Ticket (warning):** degraded but not down, slow-burn SLO, capacity trending. Address within business hours.

Every alert must have:

1. A **threshold** and **duration** (not just "is it nonzero").
2. A **runbook link** — the response doc for the alert.

## Runbook links

Every alert must link to a runbook (stored in your docs/wiki) containing:

- What this alert means (symptom, not cause).
- Triage steps.
- Escalation path.
- How to verify the alert is resolved.

## Uptime checks

Uptime checks are external synthetic probes. They answer: "can users reach us from the outside?"

- Probe every critical endpoint (login, checkout, health).
- Probe from multiple regions.
- Alert on failures and on p95 latency exceeding an SLA threshold.

## The "find bugs before customers know" checklist

- [ ] RED metrics on every request-driven service.
- [ ] USE metrics on every critical resource.
- [ ] SLOs defined for the top user journeys.
- [ ] Two-tier alerting (page + ticket) with runbooks.
- [ ] Release markers on deploys for differential analysis.
- [ ] Error-rate baselines compared against historical norms.
- [ ] Synthetic uptime checks on critical paths.
- [ ] Drills: metric spike → trace → wide event → code fix.