# Observability application stack for Kubernetes on Wodby

Deploy traces, metrics and logs in one app on Kubernetes with Wodby: the
OpenTelemetry Collector receives all three, Grafana Tempo stores traces,
VictoriaMetrics stores metrics, Grafana Loki stores logs, and Grafana shows
them together.

This repository defines the Wodby stack manifests and default service
composition for the observability stack.

<!-- wodby:generated:start -->

## Stack contract

- [Observability stack on Wodby](https://wodby.com/stacks/observability)
- [Browse Wodby application stacks](https://wodby.com/stacks)
- [Observability stack guide](https://wodby.com/docs/2.0/stacks/catalog/observability/)
- [Wodby stack documentation](https://wodby.com/docs/2.0/stacks/)
- [Stack manifest reference](https://wodby.com/docs/2.0/stacks/template/)

## Service definitions

- [Grafana Tempo service](https://github.com/wodby/service-tempo)
- [Grafana Loki service](https://github.com/wodby/service-loki)
- [VictoriaMetrics service](https://github.com/wodby/service-victoria-metrics)
- [OpenTelemetry Collector service](https://github.com/wodby/service-opentelemetry-collector)
- [Grafana service](https://github.com/wodby/service-grafana)

## What's included

| Component / service | Default configuration |
| --- | --- |
| Grafana Tempo<br>`tempo` | required; enabled by default; volumes: `data` 20 GB |
| Grafana Loki<br>`loki` | required; enabled by default; volumes: `data` 20 GB |
| VictoriaMetrics<br>`victoria-metrics` | required; enabled by default; volumes: `data` 20 GB |
| OpenTelemetry Collector<br>`otel-collector` | required; enabled by default; links: `traces` → `tempo`, `logs` → `loki`, `metrics` → `victoria-metrics` |
| Grafana<br>`grafana` | optional; enabled by default; volumes: `data` 10 GB; links: `tempo` → `tempo`, `loki` → `loki`, `victoria-metrics` → `victoria-metrics` |

Enabled optional services are selected by default but can be excluded when an
app is created. Disabled optional services are available but not selected by
default. Required services cannot be excluded.

## Validate the stack manifest

```bash
wodby stack validate-manifest stack.yml --org <org-id>
```

<!-- wodby:generated:end -->

## Send telemetry

Applications send traces, metrics and logs to the OpenTelemetry Collector over
OTLP. Its HTTP port has a public HTTPS domain, so applications in other
environments, other clusters or outside Wodby can use it. Every request needs
the Collector's generated ingest token.

Set the standard OpenTelemetry SDK variables in the application:

```
OTEL_EXPORTER_OTLP_ENDPOINT=https://<Collector domain>
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
OTEL_EXPORTER_OTLP_HEADERS=Authorization=Bearer%20<ingest token>
OTEL_SERVICE_NAME=<application name>
```

Apps from the Wodby application stacks can instead enable their own Collector
and set its upstream settings to this Collector's domain and token.

## In Grafana

Grafana has VictoriaMetrics (default), Loki and Tempo as data sources. From a
span in Tempo you can open the logs of the same trace in Loki, and a log line
with a trace ID links to the trace.
