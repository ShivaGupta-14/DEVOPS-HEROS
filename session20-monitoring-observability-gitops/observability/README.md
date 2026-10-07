# Observability

Monitoring tells us **when** something is wrong (CPU is 95%, service is down). Observability is being able to find out **why** it is wrong, just by looking at the data the system already sends out, without adding new code or logging into servers.

For that we use three types of data, called the three pillars.

## 1. Metrics

Numbers measured over time. Small and cheap to store, so we keep them for a long time and build alerts on them.

```text
node_cpu_seconds_total{mode="idle"}  12345.6
http_requests_total{status="500"}    42
up{job="node"}                       1
```

Good for: CPU, memory, request rate, error rate, latency, is the target up.
Answers: *Is something wrong? How bad? Since when?*

## 2. Logs

Text records of events with a timestamp. Much more detail than metrics, but heavy to store and search.

```text
time=2026-10-07T16:32:33Z level=ERROR msg="db connection refused" host=10.0.1.5
```

Good for: error messages, stack traces, who did what.
Answers: *What exactly happened?*

## 3. Traces

A trace follows one request through all the services it touches. Each step is a span with its own time, so we can see which service was slow.

```text
GET /checkout                    820ms
 ├─ api-gateway                   15ms
 ├─ order-service                 90ms
 └─ payment-service              700ms   <- slow one
     └─ postgres query           650ms
```

Answers: *Where in the chain is the problem?* Very useful in microservices.

## How they work together

Alert on metric (error rate went up) -> check the trace to find which service -> read that service's logs to get the actual error.

## Why observability is required

- Microservices and Kubernetes have many moving parts. Pods restart, move between nodes, scale up and down, so we can't just SSH into one server and check.
- Most failures are new ones we didn't expect, so fixed dashboards are not enough. We need data to ask new questions.
- Less downtime. Faster root cause means lower MTTR (mean time to recovery).
- Needed for SLOs: we can't promise 99.9% uptime without measuring it.

## Common tools

| Pillar | Tools |
|---|---|
| Metrics | Prometheus, Grafana, Datadog, CloudWatch, metrics-server |
| Logs | Loki, ELK (Elasticsearch, Logstash, Kibana), EFK (Fluentd/Fluent Bit), CloudWatch Logs |
| Traces | Jaeger, Grafana Tempo, Zipkin, AWS X-Ray |
| All / standard | OpenTelemetry (one standard to collect all three), Grafana LGTM stack, Datadog, New Relic |

## Kubernetes observability

- **Metrics:** `metrics-server` gives live CPU/memory for `kubectl top` and HPA. For history and alerts we use `kube-prometheus-stack` (Prometheus, Alertmanager, Grafana, node-exporter, kube-state-metrics). kube-state-metrics gives object state like desired vs ready replicas, pod restarts.
- **Logs:** containers write to stdout, `kubectl logs` reads them. In real clusters a DaemonSet (Fluent Bit / Promtail) ships logs from every node to Loki or Elasticsearch, because logs are lost when a pod is deleted.
- **Traces:** apps send spans using OpenTelemetry SDK to an OTel Collector, then to Jaeger/Tempo. Service meshes like Istio can add tracing without code changes.
- **Events and health:** `kubectl get events`, `kubectl describe`, and liveness/readiness probes tell us why a pod is not ready or keeps restarting.

```bash
kubectl top nodes
kubectl top pods -n session20
kubectl logs deployment/session20-mini -n session20
kubectl get events -n session20 --sort-by=.lastTimestamp
```
