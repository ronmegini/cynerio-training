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

### Thanos
Thanos is an open-source, highly available metric system that has been designed to work alongside Prometheus, enhancing its scalability and long-term storage capabilities. The main goal of Thanos is to allow easy access to metrics at high-availability while also enabling longer-term storage solutions beyond what vanilla Prometheus provides.

#### Key Features:
- Global Query View: Allows querying of all connected Prometheus servers, providing a unified view of metrics from different clusters or data centers.
- Unlimited Retention: With the integration of object storage (e.g., Amazon S3, Google Cloud Storage, or any S3-compatible storage), Thanos provides long-term storage capabilities beyond the local storage of Prometheus.
- Downsampling and Compression: Reduces the size and query time of historical data.
- High Availability: Achieved by replication of data and deduplication of fetched metrics.
- Scalable: Designed to scale with your needs, handling millions of time series.

#### Components:
- Sidecar: This component runs alongside a Prometheus instance. It uploads TSDB (Time Series DataBase) blocks to object storage and offers a gRPC API to access real-time metrics as well as the metrics in the remote storage.
- Store Gateway: Provides a unified access point to metrics in the object storage, making them available for querying. It's essentially the bridge between long-term storage and the real-time querying capabilities of Thanos.
- Compactor: This component is responsible for compressing, downsampling, and applying retention policies to data in the object storage. Helps in optimizing storage and improving query performance for older data.
- Querier (or Query): This is the component you query against (similar to how you'd query a Prometheus server). It aggregates the data from multiple sources like Prometheus servers, Sidecars, and Store Gateways, providing a global view of all data.
- Ruler: Provides a way to evaluate Prometheus recording and alerting rules against the global metric data, which is especially useful when you want rule evaluations to be performed on older historical data or on data from different clusters.
- Receiver: Accepts data pushed by Prometheus, useful for setups where a pull model is not feasible. Can replicate data for HA setups.
- Frontend: Introduced for improved query performance. Splits and parallelizes larger query jobs, reducing the overall query time.
- Tools: Thanos provides various tools for inspecting and modifying blocks in object storage, useful for maintenance and troubleshooting.
