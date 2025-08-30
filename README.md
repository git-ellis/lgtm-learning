# Local Observability Stack: FastAPI + Grafana + Loki + Tempo + Prometheus

This repo provides a minimal, up-to-date observability stack you can run locally with Docker Compose:

- FastAPI app with:
  - OpenTelemetry traces exported via OTLP HTTP to Tempo
  - Prometheus metrics at `/metrics`
  - Logs to stdout with trace_id/span_id correlation (scraped by Promtail -> Loki)
- Grafana pre-provisioned with data sources for Prometheus, Loki, and Tempo, including log-to-trace linking
- Loki for logs, Tempo for traces, Prometheus for metrics

## Prerequisites
- Docker + Docker Compose

## Start

```bash
docker compose up -d --build
```

Services after startup:
- App: http://localhost:8000 (routes: `/`, `/work`, `/metrics`)
- Grafana: http://localhost:3000 (login: `admin` / `admin`)
- Prometheus: http://localhost:9090
- Loki API: http://localhost:3100
- Tempo API: http://localhost:3200

## Try it
- Hit the app a few times:
  ```bash
  curl http://localhost:8000/
  curl http://localhost:8000/work
  ```
- Metrics in Grafana Explore → Prometheus: search for metrics (e.g., type `http` or `fastapi`) and plot request counts/latency.
- Logs in Grafana Explore → Loki: select `{service="app"}`; log lines include `trace_id=...` with a link to the trace (via Tempo)
- Traces in Grafana Explore → Tempo: search by service `fastapi-demo` or paste a trace_id

## Notes
- Images use `:latest` tags per request to track latest releases.
- Python deps are installed with `uv` in the app image for fast, cached builds.
- Trace/log correlation uses env `OTEL_PYTHON_LOG_CORRELATION=true` and a log format injecting `trace_id`/`span_id`.
- Traces are sent OTLP/HTTP to an OpenTelemetry Collector, then exported to Tempo.
- Prometheus scrapes the app directly; no OTel metrics exporter is used.

## Tear down
```bash
docker compose down -v
```
