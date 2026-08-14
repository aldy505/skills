# OpenTelemetry Quickstart

OpenTelemetry (OTel) is the vendor-neutral instrumentation framework for traces, metrics, and logs. All major backends (DataDog, Grafana, Honeycomb, New Relic, Sentry) accept OTel data via OTLP exporters.

## Auto-instrumentation

Auto-instrumentation captures outbound HTTP calls, DB queries, message queue operations, and framework-level spans (HTTP server requests) with zero code changes.

**Python (Python 3.12+):**

```bash
pip install opentelemetry-distro opentelemetry-exporter-otlp
opentelemetry-bootstrap -a install
```

```bash
# Run with auto-instrumentation
opentelemetry-instrument python app.py
```

**Node.js:**

```bash
npm install @opentelemetry/auto-instrumentations-node
```

```js
// register-instrumentations.js
const { NodeSDK } = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');

const sdk = new NodeSDK({
  instrumentations: [getNodeAutoInstrumentations()],
});
sdk.start();
```

## Manual span creation

For custom spans inside business logic:

```python
from opentelemetry import trace

tracer = trace.get_tracer("my-service")

with tracer.start_as_current_span("process-order") as span:
    span.set_attribute("order.id", order_id)
    span.set_attribute("order.item_count", len(items))
    # ... business logic ...
    span.add_event("validation_passed")
```

## Span attributes → wide events

Attributes on a span are the wide event. Add attributes from every relevant context:

```python
# infra
span.set_attribute("deployment.version", os.getenv("APP_VERSION"))
span.set_attribute("k8s.pod.name", os.getenv("HOSTNAME"))

# user
span.set_attribute("user.id", user.id)
span.set_attribute("user.region", user.region)

# business
span.set_attribute("order.total_usd", order.total)
span.set_attribute("cart.item_count", order.item_count)

# feature flags
span.set_attribute("flag.new_checkout", rollout.is_enabled("new_checkout", user))
```

Use attribute **naming conventions** (`namespace.action`, dotted notation) consistently across services.

## Context propagation

Context propagation passes trace/span IDs across async boundaries (HTTP, message queues, background workers) so a single trace spans multiple services.

**HTTP (already handled by auto-instrumentation):** OTel propagates `traceparent` and `tracestate` headers automatically.

**Message queues (manual):**

```python
from opentelemetry import propagate, trace

# In the producer: inject context into message headers
def publish(message, queue):
    headers = {}
    propagate.inject(dict, headers)  # injects traceparent
    queue.publish(message, headers=headers)

# In the consumer: extract context from message headers
def consume(raw_message):
    ctx = propagate.extract(raw_message.headers)  # extracts traceparent
    with tracer.start_as_current_span("process-message", context=ctx):
        handle(raw_message.body)
```

**Async boundaries (e.g., Python asyncio, JS Promises):** the context is per-thread; use `context.attach()` / `context.detach()` if crossing threads or task boundaries explicitly.

## Head-based sampling

The default sampling strategy: at ingress, decide probabilistically whether to keep a trace. Every child span inherits the parent's decision.

- Simple; no collector needed.
- **Problem:** may discard errors and slow requests by accident.

```python
from opentelemetry.sdk.trace.sampling import ParentBasedTraceIdRatio

# keep ~50% of traces
sampler = ParentBasedTraceIdRatio(rate=0.5)
```

## Tail-based sampling

Buffer spans before deciding; always keep traces with errors or high latency.

- Requires an OTel **Collector** to accumulate spans.
- Guarantees all error/slow traces are retained; samples only normal-path traffic.

```yaml
# otel-collector-config.yaml
processors:
  tail_sampling:
    decision_wait: 5s
    policies:
      - name: keep-errors
        type: status_code
        status_code: { status_codes: [ERROR] }
      - name: keep-slow
        type: latency
        latency: { threshold_ms: 2000 }
      - name: sample-rest
        type: probabilistic
        probabilistic: { sampling_percentage: 10 }
```

Use tail-based sampling when budget allows buffering (high-volume services) and complete error visibility is critical.

## Verifying it works

1. Start your app with instrumentation (auto or manual).
2. Generate a trace: make an HTTP request, process a message, trigger an error.
3. Check the OTel Collector / backend UI for the trace.
4. Confirm attributes appear on the span.
5. Confirm trace ID links related spans across services.