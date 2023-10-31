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

### Installation
Deploy Prometheus with Thanos Sidecar:
```
# Add the Prometheus community Helm chart repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus with Thanos sidecar configuration
helm install prometheus prometheus-community/prometheus \
--set thanos.create=true \
--set thanos.objstoreConfig=<path_to_your_objstore_config.yaml>
```
Deploy Thanos Components:
```
helm install thanos-query prometheus-community/thanos \
--set query.enabled=true \
--set query.http.enabled=true \
--set query.dnsDiscovery.enabled=false
```
Deploy Grafana:
```
# Add the Grafana Helm chart repository
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install Grafana
helm install grafana grafana/grafana
```

### PromQL
Prometheus Query Language, is the powerful and flexible query language used by Prometheus to retrieve and work with stored metric data. With PromQL, we can aggregate and analyze time series data in various ways, allowing us to gain insights, generate alerts, and create comprehensive visualizations via tools like Grafana.

#### Basic Concepts:
- Time Series: A time series in Prometheus contains a set of timestamp/value metrics for a single metric name and label set.
- Labels: Time series are identified by metric names and sets of key-value pairs (labels). For example, http_requests_total{method="GET", handler="/api"}.
- Samples: Each time series consists of individual data points known as samples. Each sample contains a float64 value and a millisecond-precision timestamp.

#### Metric Types:
- Counter: A cumulative metric that represents a single monotonically increasing counter. It only increases or gets reset to zero.
- Gauge: A metric that can go up or down, representing a single numerical value that can arbitrarily increase or decrease.
- Histogram: Samples observations (like request durations or response sizes) and counts them in configurable buckets.
- Summary: Similar to a histogram, but also provides a set of quantiles (e.g., 50th, 90th, and 99th percentile).

#### Basic Operators and Functions:
- Arithmetic: +, -, *, /, %, ^.
- Comparison: ==, !=, >, <, >=, <=.
- Logical: and, or, unless.
- Functions: rate(), sum(), avg(), max(), min(), histogram_quantile(), and many others.

#### Common Query Patterns:
- Instant Vector Selector: This is a basic query for fetching data.
`http_requests_total{handler="/api"}`: Returns all the time series with the metric name http_requests_total and the label handler set to /api.
- Range Vector Selector: Fetches data over a period.
`http_requests_total[5m]`: Returns the http_requests_total metric over the last 5 minutes.
- Rate over a Range Vector: Computes the per-second rate over a period.
`rate(http_requests_total[5m])`: Returns the per-second rate of http_requests_total over the last 5 minutes.
- Aggregation: Aggregates data across multiple time series.
`sum(http_requests_total)`: Sums up all the time series data for http_requests_total.
`avg(http_requests_total) by (method)`: Averages the data, grouped by the method label.
- Operators with Vectors: Combine different time series or scalar values.
`http_requests_total / http_request_duration_milliseconds`: Divides the total requests by the total duration, giving an average request duration.
- Histogram and Quantiles: Generate quantiles from histogram data.
`histogram_quantile(0.95, sum(rate(http_request_duration_bucket[5m])) by (le))`: Returns the 95th percentile of request durations over the past 5 minutes.
