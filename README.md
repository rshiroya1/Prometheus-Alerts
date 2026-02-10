# prometheus-alert-pack

SRE fundamentals in one place: **Prometheus alert rules** for high latency, pod restarts, and error rate.

## What it shows

| Area           | Alerts |
|----------------|--------|
| **High latency** | Request latency (p95 / average) above threshold; critical when p95 > 3s |
| **Pod restarts** | Pods restarting frequently; critical when in a restart loop |
| **Error rate**   | 5xx error rate above 5% (warning) or 10% (critical) |

## Files

```
prometheus-alert-pack/
├── alerts.yaml   # Alert rules (Prometheus format)
└── README.md     # This file
```

## Alert rules

- **High latency**  
  - `HighLatency`: p95 > 1s or average > 0.5s for 5m → `severity: warning`  
  - `HighLatencyCritical`: p95 > 3s for 2m → `severity: critical`

- **Pod restarts**  
  - `PodRestartingFrequently`: > 3 restarts in 1h → `severity: warning`  
  - `PodRestartLoop`: restart rate indicates a loop → `severity: critical`

- **Error rate**  
  - `HighErrorRate`: 5xx rate > 5% for 5m → `severity: warning`  
  - `HighErrorRateCritical`: 5xx rate > 10% for 2m → `severity: critical`

## Metrics assumed

Rules use common names; adjust to match your exporters:

| Rule focus   | Metrics used |
|-------------|--------------|
| Latency     | `http_request_duration_seconds_bucket`, `_sum`, `_count` (or your histogram) |
| Pod restarts| `kube_pod_container_status_restarts_total` (from kube-state-metrics) |
| Error rate  | `http_requests_total` with label `status=~"5.."` (or your request counter) |

If your metrics differ (e.g. `request_duration_seconds`, `istio_request_duration_milliseconds`), change the metric names in `alerts.yaml` and keep the same structure.

## How to use

1. **Prometheus config** — add the rule file:

   ```yaml
   rule_files:
     - /path/to/alerts.yaml
   ```

2. **Prometheus Operator (Kubernetes)** — use a `PrometheusRule`:

   ```yaml
   apiVersion: monitoring.coreos.com/v1
   kind: PrometheusRule
   metadata:
     name: sre-alert-pack
   spec:
     groups: []   # paste contents of the `groups` section from alerts.yaml
   ```

3. Reload Prometheus (or let the operator sync), then check **Alerts** in the Prometheus UI.

## Requirements

- Prometheus 2.x
- For pod alerts: **kube-state-metrics** and metrics from your apps/ingress (e.g. histogram and request counters).

## License

Use and adapt as you like.
