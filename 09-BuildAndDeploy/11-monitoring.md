# Application Monitoring

[Back to guide index](README.md)

## 11.1 Monitoring Objectives

Monitoring should tell you:

- Is the service up?
- Is it healthy?
- Is it fast enough?
- Is it erroring?
- Is it resource-constrained?
- Are dependencies failing?

## 11.2 Health Check Endpoints

Expose simple health endpoints.

Typical paths:

- `/health`
- `/ready`
- `/live`

Design guidance:

- Liveness should be cheap and minimal
- Readiness may validate critical dependencies
- Avoid overly expensive health logic

Example response:

```json
{
  "status": "ok",
  "version": "1.4.2",
  "commit": "abc1234"
}
```

## 11.3 Metrics to Track

Core metrics:

- Request rate
- Error rate
- Response latency
- CPU usage
- Memory usage
- File descriptor usage
- Queue depth
- Database connection pool usage

## 11.4 Prometheus

Prometheus scrapes metrics endpoints.

Typical endpoint:

```text
/metrics
```

Best for:

- Time-series metrics
- Alerting rules
- Service dashboards

## 11.5 Grafana

Grafana visualizes metrics from Prometheus and other sources.

Useful dashboard panels:

- HTTP status rates
- P95 latency
- JVM heap usage
- Node event loop lag
- Go GC pause duration
- Python worker restarts
- .NET request duration

## 11.6 APM Tools

Commercial APM tools include:

- New Relic
- Datadog
- Dynatrace
- AppDynamics

They often provide:

- Distributed tracing
- Service maps
- Error monitoring
- Deployment markers
- Runtime-specific insights

## 11.7 Prometheus vs APM

| Capability | Prometheus | APM Platforms |
|---|---|---|
| Metrics | Excellent | Excellent |
| Tracing | Limited unless paired | Strong |
| Logs | External integration | Often integrated |
| Cost | Often lower self-hosted | Usually commercial |
| Depth by runtime | Depends on exporters | Often richer |

## 11.8 Log Management

Logs should be:

- Structured where possible
- Centralized
- Searchable
- Retained according to policy
- Correlated with requests and traces

Popular stacks:

- ELK/Elastic Stack
- OpenSearch
- Loki + Grafana
- Splunk

## 11.9 Logging Best Practices

Include fields such as:

- Timestamp
- Level
- Service name
- Hostname
- Environment
- Request ID
- Trace ID
- User ID where appropriate and compliant

## 11.10 Error Tracking with Sentry

Sentry is useful for:

- Capturing exceptions
- Grouping errors
- Tracking releases
- Alerting on regressions

Typical usage:

- Include release version
- Include environment name
- Avoid sending sensitive data

## 11.11 JVM Monitoring

Key JVM metrics:

- Heap used/max
- GC pause time
- Thread count
- Class loading
- CPU usage
- Request latency

Useful tools:

- JMX exporters
- Micrometer
- Java Flight Recorder

## 11.12 Python Monitoring

Useful metrics:

- Gunicorn worker count
- Worker restarts
- Response time
- Queue time
- Memory growth over time
- Dependency latency

## 11.13 Node.js Monitoring

Useful metrics:

- Event loop lag
- Heap usage
- Garbage collection pauses
- Open handles
- Request latency
- WebSocket connection count

## 11.14 Go Monitoring

Useful metrics:

- Goroutine count
- GC duration
- Heap allocation
- Request latency
- Error rate
- Dependency latency

## 11.15 .NET Monitoring

Useful metrics:

- Request duration
- GC collections
- Heap size
- Thread pool saturation
- Exceptions per minute

## 11.16 Alerting Recommendations

Alert on symptoms and saturation, not only uptime.

Examples:

- 5xx rate above threshold
- P95 latency above SLO threshold
- Memory near limit with upward trend
- Connection pool exhaustion approaching critical levels

## 11.17 Deployment Markers

Record deployments in monitoring tools.

This helps correlate:

- Error spikes
- Latency regressions
- Resource behavior changes

## 11.18 Common Monitoring Gaps

- No readiness checks
- No application-level metrics
- No version metadata in health endpoints
- Logs without correlation IDs
- Alerts that are too noisy or too weak

---

---

# Appendix F. Production Readiness Checklists

## F.1 Universal Build Checklist

- Source from trusted repository
- Pinned dependencies
- CI reproducibility
- Tests green
- Artifact versioned
- Artifact checksum recorded
- SBOM generated if required

## F.2 Universal Deployment Checklist

- Target host prepared
- Service user exists
- Config present
- Secret access working
- Health checks defined
- Monitoring connected
- Rollback plan ready
- Last known good release retained

## F.3 Universal Runtime Checklist

- Runs as non-root
- Reverse proxy secured
- TLS enabled
- Logs centralized
- Metrics exposed
- Restart policy set
- Resource limits reviewed
- Alerting configured

---
