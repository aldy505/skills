# Signals — Overlap, Tie-breakers, and Code Snippets

## Signal relationships

| Overlap | When to use this one |
|---------|----------------------|
| Span attribute vs metric | Attribute: high cardinality, rich detail, not pre-aggregated. Metric: time-series dashboards, thresholds, SLOs. |
| Log vs span | Log: audit trail, human-readable event, external system output. Span: request-path instrumentation, timing, trace-linked context. |
| Log vs metric | Log: narrative context, variable detail. Metric: aggregate over time, alerting thresholds, dashboard. |
| Error vs log | Error: counts for alerting and SLOs. Log: the narrative explaining why it happened (or a span event for structured exception capture). |

## Wide events and the other signals

A wide event (span with rich attributes) is the default for a new per-request instrument. Use:

- **Spans for timing** — start/end, status, events for errors; attributes hold context.
- **Metrics for trends** — counts/rates from span data; SLO burn/alerts.
- **Logs for narrative** — only when the span is not enough (audit, human narration, external output).
- **Events on spans** — for exception capture or discrete moments inside a span.

Do not put the same detail in both a log line and a span attribute; span attributes win. A log that reproduces what a span already carries is noise.

## Retention and sampling

| Signal | Typical retention |
|--------|-------------------|
| Metrics | 90–390 days (pre-aggregated, cheap) |
| Spans (wide events) | 15–30 days (rich, expensive) |
| Logs | 30–90 days (varies by team/compliance) |
| Errors | 90–365 days (triage + SLO history) |

- **Head-based sampling** — probabilistic at ingress; keeps simple traffic within budget; discards some errors and slow requests by accident.
- **Tail-based sampling** — buffer spans, then decide after trace completion; always keep errors and slow traces; use with a collector.
- **Never sample metrics** — always compute.

## Code snippets

**Python — OpenTelemetry span (wide event):**

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider

tracer = trace.get_tracer("order-service")

def handle_request(req):
    with tracer.start_as_current_span("POST /orders") as span:
        span.set_attribute("http.method", req.method)
        span.set_attribute("http.status_code", 200)
        span.set_attribute("user.id", req.user.id)
        span.set_attribute("order.id", req.order.id)
        span.set_attribute("order.item_count", len(req.items))
        # event: discrete moment inside the span
        span.add_event("discount_applied", {"discount.code": "WELCOME"})
```

**TypeScript — OpenTelemetry span with attributes:**

```typescript
import { trace, SpanStatusCode } from '@opentelemetry/api';

const tracer = trace.getTracer('order-service');

export async function handleRequest(req: Request) {
  return tracer.startActiveSpan('POST /orders', async (span) => {
    span.setAttribute('http.method', req.method);
    span.setAttribute('http.status_code', 200);
    span.setAttribute('user.id', req.userId);
    span.setAttribute('order.id', req.orderId);
    try {
      // ... business logic ...
    } catch (err) {
      span.setStatus({ code: SpanStatusCode.ERROR, message: (err as Error).message });
      throw err;
    } finally {
      span.end();
    }
  });
}
```

**Python — metric from span data (OTel SDK or pipeline aggregation):**

```python
from opentelemetry.metrics import get_meter_provider

meter = get_meter_provider().get_meter("order-service")
request_counter = meter.create_counter(
    "http.server.requests",
    description="Total HTTP server requests",
)

def record_request(method, status):
    request_counter.add(1, {"http.method": method, "http.status_code": status})
```

**TypeScript — emit a metric:**

```typescript
import { metrics } from '@opentelemetry/api';

const meter = metrics.getMeter('order-service');
const requestCounter = meter.createCounter('http.server.requests', {
  description: 'Total HTTP server requests',
});

export function recordRequest(method: string, status: number) {
  requestCounter.add(1, { 'http.method': method, 'http.status_code': status });
}
```