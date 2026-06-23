Step 9: Monitoring & Logging
When you move to highly distributed systems, you cannot just log into a single server to see what went wrong. You need centralized tools to understand the health and behavior of your software.

1. Logging Events & Monitoring Metrics
   System visibility relies on two distinct concepts:

Metrics (Monitoring): Numeric data measured over intervals of time to assess overall system health. Examples include CPU utilization, memory usage, request counts per second, and error rate percentages. Tools like Prometheus and Grafana track these.

Logs (Event Tracking): Text strings emitted when specific events happen inside the code (e.g., [14:22:05] User 482 failed to checkout due to expired token). Tools like the ELK Stack (Elasticsearch, Logstash, Kibana) or Grafana Loki collect logs from all servers into a single searchable dashboard.

2. Anomaly Detection
   With hundreds of services emitting gigabytes of metrics and logs every hour, humans cannot watch dashboards manually. Anomaly Detection applies algorithmic rules or basic machine learning to establish a "baseline" of normal system behavior.

Alerting: If traffic suddenly drops by 80%, or if error rates spike past a standard threshold, the anomaly detection engine flags it immediately and pages the engineering team (via tools like PagerDuty or Slack alerts) before customers even realize something broke.

```text
┌────────────────────────┐ True           ┌────────────────────────┐
│ Metric crosses defined │ ─────────────► │ Trigger Severity Alert │
│ operational threshold  │                │ (PagerDuty / Slack)    │
└────────────────────────┘                └───────────┬────────────┘
            │                                         │
            │ False                                   ▼
            ▼                             ┌────────────────────────┐
┌────────────────────────┐                │ On-Call Engineer pages │
│ Engine stays quiet;    │                │ to fix the microservice│
│ logs standard baseline │                │ before users notice.   │
└────────────────────────┘                └────────────────────────┘

- **Threshold Breeches:** If your production API error count crosses a configured threshold (e.g., spiking over `1%` of total traffic), or if network traffic suddenly drops by an unusual `80%` on a busy afternoon, the engine instantly flags the anomaly.
- **Incident Paging:** The detection pipeline automatically packages the context data and routes high-priority pages to on-call engineers via developer operations communication hubs (like **PagerDuty**, **Opsgenie**, or dedicated **Slack webhooks**). This notifies the team to remediate the service health issues before customers face application failures.
```
