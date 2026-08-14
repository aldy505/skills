# Wide Events

## The "ask any question" framing

Traditional telemetry (metrics, standard logs) answers the questions you thought to ask up front. Wide events let you ask questions you didn't anticipate after the fact — because every meaningful step of every request is captured, in full context, queried at analysis time instead of at write time.

From Alok Singh's "A Crash in the Stream": the debugging power comes from being able to slice by any dimension — a customer, a region, a flag value, a dependency, a code path — on stored data, with no new instrumentation.

Example questions wide events answer:

- Which user(s) hit this error, and what were they doing?
- What feature-flag combination correlates with the failure?
- Which region/version of the dependency was in the failing path?
- What was the auth context, ID, region, and upstream latency for this one request?
- Which slice of traffic experienced latency > 2s that metrics smoothed over?

## Characteristics

- **High cardinality** — values are unique per request (request IDs, user IDs, hostnames), not bucketed ahead of time.
- **High dimensionality** — dozens to hundreds of attributes per event.
- **Context-rich** — infra, HTTP, user, business, dependencies, flags, errors all on one event.
- **Emitted once per request (or per hop)** — one complete record, not many partial ones.
- **Connected by trace/request ID** — events link into a trace for follow-the-path debugging.

## Categories of attributes to include

| Category | Example attributes |
|----------|--------------------|
| Infrastructure / deploy | service, host, cluster, region, version, release marker, pod/instance |
| HTTP | method, path, status, protocol, duration, user-agent, referer |
| User / customer | user_id, account_id, plan, region, session_id |
| Business / domain | order_id, product_id, package, mode, machine, outcome |
| Performance / dependencies | DB latency, upstream latency, queue size, retries, cache hit/miss |
| Feature flags | flag, variation, rollout group |
| Error context | error type, message, code, stack, span status |

## Before / after

**Before — a single deep log line (structured log):**

```json
{"time":"...","level":"error","msg":"checkout failed","order":"9384712","latency_ms":1803}
```

Two attributes, one error path. To ask "which flag was on", "which region", "was the payment upstream slow", there is nothing to slice by.

**After — one wide event per request:**

```json
{
  "type":"http_request",
  "service":"checkout-api","region":"eu-west-1","version":"1.4.2","release":"r207",
  "method":"POST","path":"/checkout","status":500,"duration_ms":1803,
  "user_id":"u_43192","plan":"pro","user_region":"us-east-2",
  "order_id":"9384712","items":3,"currency":"USD",
  "payment_upstream_ms":1662,"payment_retries":1,
  "flag_new_payment_gateway":true,
  "error_type":"GatewayTimeout","error_msg":"upstream payment timeout",
  "trace_id":"4bf92f3577b34da6a3ce929d0e0e4736","request_id":"req_9f3a"
}
```

Now the question "did `flag_new_payment_gateway` cause the 500s for `eu-west-1` pro users?" is a group-by on stored data.

## Implementation patterns

**Crude pattern — accumulate a dictionary, flush in `finally`:**

```python
import contextlib

@contextlib.contextmanager
def wide_event(service, **init_attrs):
    attrs = dict(init_attrs)
    try:
        yield attrs
    finally:
        emit(attrs)   # flush once per request

# usage
with wide_event("checkout-api", user_id=user.id, region=region,
                order_id=order.id, flag_gateway=new_gateway) as ev:
    ev["status"] = 500
    ev["payment_upstream_ms"] = 1662
```

**Standard pattern — an OpenTelemetry span; every attribute lands on the span:**

```python
from opentelemetry import trace

tracer = trace.get_tracer("checkout")
with tracer.start_as_current_span("POST /checkout") as span:
    span.set_attribute("user.id", user.id)
    span.set_attribute("order.id", order.id)
    span.set_attribute("payment.upstream_ms", 1662)
    span.set_attribute("flag.new_payment_gateway", True)
    span.set_attribute("http.response.status_code", 500)
```

The span is the wide event; its trace ID connects the event to all other hops.

## Tooling requirements

- Queryable across **any dimension** (attribute filtering/grouping), not just pre-defined metric series.
- **No pre-aggregation** required to answer arbitrary slices.
- **Fast** — interactive analysis over high-cardinality data.
- **Affordable** at the intended event volume.

Honeycomb (and wide-schema backends like it) are built for this; a metrics-only backend (pre-aggregated counters) cannot answer arbitrary slice questions.

## Misconceptions

- Wide events do **not** replace infra metrics — you still need dashboards, SLO burn/trend views, and alerting that aggregates over time.
- They are **not only for outages** — they power daily product/business questions too.
- They are **not the same as a 5-field structured log** — that log is a narrow record; a wide event is a per-request, high-dimensional record.
- OpenTelemetry is **one implementation, not the only one** — the concept predates and outlives any particular SDK or vendor.
- Structured logs are **not necessarily wide events** — width (many dimensions) is the defining property, not the JSON formatting.