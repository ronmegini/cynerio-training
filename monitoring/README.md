## Monitoring

### Overview
Prometheus and Grafana are popular metric monitoring solution to monitor cloud-native applications.

### Prometheus
Prometheus is an open-source systems monitoring and alerting toolkit. It's especially good for monitoring cloud-native applications, often in conjunction with a container orchestration platform like Kubernetes. It collect the metrics, store them (like a DB), and allow to query the data.

#### Key Features:
- Pull-Based: Prometheus "pulls" metrics from the targets it monitors.
- Service Discovery: Automatically discovers services to monitor in dynamic environments like Kubernetes.
- Flexible Query Language: Uses PromQL, a flexible query language, to allow for slicing and dicing of collected data.
- Multi-dimensional Data Model: Metrics are identified by a metric name and key/value pairs (labels).
- Storage: Built-in local storage, but it can also integrate with remote storage systems. Prometheus stores time series data in its built-in TSDB. Metrics are retained for a specified period, after which they're discarded to free up space.

#### Components:
- Targets: Any entity you might want to monitor. They expose an HTTP endpoint (usually /metrics) from which Prometheus scrapes metrics.
- Exporter: Bridges the gap between Prometheus and software that doesn't export metrics in the Prometheus format. For example, node_exporter provides host-related metrics.
- Pushgateway: For supporting short-lived jobs, since Prometheus is primarily pull-based.
- Alertmanager: Manages alerts, deduplicates, groups, and routes them to the right receiver (e.g., email, Slack).

### Grafana
Grafana is an open-source platform for monitoring and observability, which can visualize data from a variety of data sources, including Prometheus.

#### Key Features:
- Dashboards: Offers visually rich, interactive, and customizable dashboards.
- Data Source Integration: Supports multiple sources, allowing for mixing several databases on one dashboard.
- Alerting: Provides visually-rich alerts.
- Annotations: Allows for correlating data with events.
- Authentication & Security: Grafana supports various methods for user authentication, including local users, OAuth, LDAP, etc.
- Plugins: Allows extension with third-party plugins for new data sources, panels, and other integrations.

#### How It Works with Prometheus:
- Grafana fetches metrics from Prometheus using PromQL.
- The fetched metrics can be visualized in various forms like graphs, tables, and heatmaps.
- Grafana dashboards are often shared in the community, so you don't always have to build from scratch.
