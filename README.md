# Observability

This repository contains hands-on examples and learning materials for Prometheus monitoring and observability.

## Course Information

Attended hands-on session from Udemy: [Prometheus Course](https://amdocsglobal.udemy.com/course/prometheus-course)  
 (Learn A to Z of Prometheus & Grafana from Basic to ADVANCE level; Complete Guide to Master DevOps Monitoring & Alerting)

This course provides a clear picture for beginners on Prometheus.

## EC2 Setup

I created an EC2 instance from AWS for my hands-on session.

### Additional Configurations for EC2 Setup

1. **Security Group Configuration**: Create inbound rules in the security group of EC2 to access all the UIs:
   - Prometheus (port 9090)
   - Python Apps for Gauge, Counter, Histogram, Summary metric types
   - Go Apps for Gauge, Counter, Histogram, Summary metric types
   - Node Exporter (port 9100)

2. **Alternative**: Use AWS Client VPN for secure access instead of opening ports to the public internet.

## Project Structure

### Folders
1. **prometheus-3.8.1.linux-amd64**: Contains Prometheus server and configuration. Download package from link https://github.com/prometheus/prometheus
   - Prometheus UI: `http://localhost:9090`
   - Configuration: `prometheus.yml`

2. **node_exporter-1.10.2.linux-amd64**: Contains the Node Exporter for Linux machines
   - Port: 9100
   - Access: `http://localhost:9100/metrics`

3. **prom_python_app**: Contains simple Python applications with client libraries to instrument Python applications for each metric type
   - Counter: `counter.py` (port 8001)
   - Gauge: `gauge.py` (port 2001)
   - Histogram: `histogram.py`
   - Summary: `summary.py` (port 3001)

4. **prom_go_app**: Contains Go applications with Prometheus client libraries demonstrating different metric types
   - Counter: `counter.go` (port 5000)
   - Gauge: `gauge.go` (port 6000)
   - Histogram: `histogram.go` (port 7000)
   - Summary: `summary.go` (port 9000)

5. **alertmanager**: Contains alert manger confuguration. Donwload from link:https://github.com/prometheus/alertmanager


## Port Mappings

| Service | Port | Endpoint |
|---------|------|----------|
| Prometheus | 9090 | `http://localhost:9090` |
| Node Exporter | 9100 | `http://localhost:9100/metrics` |
| Python Counter | 8001 | `http://localhost:8001/birthday/{name}` |
| Python Gauge | 2001 | `http://localhost:2001/birthday/{name}` |
| Python Summary | 3001 | `http://localhost:3001/birthday/{name}` |
| Go Counter | 5000 | `http://localhost:5000/birthday/{name}` |
| Go Gauge | 6000 | `http://localhost:6000/birthday/{name}` |
| Go Histogram | 7000 | `http://localhost:7000/birthday/{name}` |
| Go Summary | 9000 | `http://localhost:9000/birthday/{name}` |
| AlertManager | 9093 | `http://localhost:99093/birthday/{name}` |
All applications expose metrics at `/metrics` endpoint.

## Setup Instructions

### Python Applications

1. Install Python dependencies:
   ```bash
   pip install prometheus_client
   ```

2. Run the applications:
   ```bash
   python counter.py
   python gauge.py
   python summary.py
   python histogram.py
   ```

### Go Applications

1. Install Go (version 1.9 or higher):
   - Download from: https://go.dev/doc/install
   - Or install via: `sudo yum install golang`

2. Initialize Go module and install dependencies:
   ```bash
   cd prom_go_app
   go mod init prom_go_app
   go mod tidy
   ```

3. Run the applications:
   ```bash
   # Counter
   nohup go run counter.go > counter_go.log 2>&1 &
   
   # Gauge
   nohup go run gauge.go > gauge_go.log 2>&1 &
   
   # Histogram
   nohup go run histogram.go > histogram_go.log 2>&1 &
   
   # Summary
   nohup go run summary.go > summary_go.log 2>&1 &
   ```

### Prometheus Configuration

The `prometheus.yml` file is configured to scrape all the applications running on localhost. Update the targets if running on different hosts or using private IPs.


## Metric Types

### Counter
- Monotonically increasing metric
- Example: Total number of HTTP requests
- Use case: Request rate, error count

### Gauge
- Value that can go up or down
- Example: Number of requests in progress, current memory usage
- Use case: Current state metrics

### Histogram
- Samples observations and counts them in configurable buckets
- Example: Request latency distribution
- Use case: Response time percentiles (p50, p95, p99)

### Summary Metric
- Similar to Histogram, but calculates quantiles over a sliding time window
- **Example**: In `summary.py` and `summary.go`, the metric calculates response time (difference between end and start time)
- **Average Latency Calculation**: Divide `rate(latency_sum)` by `rate(latency_count)`
- Use case: Pre-calculated quantiles for latency

## Prometheus Queries

### Example Queries

- **Request Rate**: `rate(go_app_requests_count[5m])`
- **Average Latency (Summary)**: `rate(go_app_response_latency_seconds_sum[5m]) / rate(go_app_response_latency_seconds_count[5m])`
- **95th Percentile Latency (Histogram)**: `histogram_quantile(0.95, rate(go_app_response_latency_seconds_bucket[5m]))`
- **Requests in Progress**: `go_app_requests_inprogress`

## Resources

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Prometheus Client Libraries](https://prometheus.io/docs/instrumenting/clientlibs/)
- [Go Prometheus Client](https://github.com/prometheus/client_golang)
- [Python Prometheus Client](https://github.com/prometheus/client_python)


## What to Instrument?

Instrumentation falls into 2 main categories: **Services** and **Libraries**.

### Service Instrumentation

Service instrumentation can be broken into 3 types: **Online**, **Offline**, and **Batch Jobs**.

#### Online Services
- **Description**: Systems or persons are waiting for a response
- **Key Metrics**:
  - Request Rate (Counter)
  - Latency (Summary or Histogram)
  - Error Rate (Counter)
  - In-progress Requests (Gauge)
- **Best Practice**: Monitor both on client and service side

#### Offline Services
- **Description**: No one actively waiting for response; batching is common with multi-step processing (e.g., log processing)
- **Key Metrics**:
  - Items coming in
  - Ingress items
  - Items sent out
  - Errors

#### Batch Jobs
- **Description**: Similar to Offline systems; main difference is that offline runs continuously while batch jobs run at defined schedules
- **Key Metrics**:
  - How long a job took
  - The runtime
  - Time at which job completed (failed or success)
- **Note**: Push gateways should be used for this type

### Libraries

Within each service, we have libraries.

1. **Heavy Libraries**: Track internal errors and latency time within the library
2. **External Access Libraries**: Track query count and latency when the library is used to access outside information

### Rules 
There are two types of rules in Prometheus: **Recording Rules** and **Alerting Rules**.

#### Recording Rules
- **Purpose**: Precompute commonly used expressions and save results in new time series.
- Go through prometheus-3.8.1.linux-amd64/rules/rule1.yml to view sample rules created
- **Implementation**:
  - Written in a separate rule file in YAML format.
  - Write a PromQL expression to evaluate regularly.
  - Give each record a name; instead of using the full PromQL, one can use the name.
  - Feed the file to Prometheus service via `prometheus.yml` configuration.
  - Prometheus will evaluate regularly after a specified time interval and store as a new time series and metric name in its storage engine.
- **Benefits**: 
  - Faster execution
  - Useful in speeding up dashboards
  - Can be used by anyone using APIs
  - Reduces load on Prometheus query engine

#### Alerting Rules
- **Purpose**: Define conditions that should trigger alerts based on metric values.
- **Implementation**:
  - Run ./alertmanager to bring up the UI on port 9093
  - Define logic in the form of PromQL expressions to evaluate continuously.
  - When conditions are met, they become alerts.
  - Example: CPU usage should not be more than 90%.
  - Written in YAML format (similar to recording rules).
  - Each alert has:
    - **Alert name**: Unique identifier for the alert
    - **Expr**: PromQL expression that evaluates to true when alert should fire
    - **For**: Duration the condition must be true before alert fires (optional)
    - **Labels**: Additional labels to attach to the alert (e.g., severity, team)
    - **Annotations**: Human-readable information about the alert (summary, description)
  - Feed the file to Prometheus service via `prometheus.yml` configuration.
  - Prometheus evaluates alerts regularly and sends them to Alertmanager when conditions are met.
- **Alertmanager Integration**:
  - Prometheus sends alerts to Alertmanager service.
  - Alertmanager handles routing, grouping, silencing, and notification of alerts.
  - Supports multiple notification channels (email, Slack, PagerDuty, webhooks, etc.).
- **Benefits**:
  - Proactive monitoring and issue detection
  - Centralized alert management
  - Alert grouping and deduplication
  - Flexible notification routing
  - Reduces alert fatigue through intelligent grouping

##### Routing Alerts
Alertmanager uses the concept of **routing** to determine how alerts are processed and sent to the right receiver depending on their labels. The routing tree is defined in `alertmanager.yml` under the `route:` section.

**How Routing Works:**
- The top-level `receiver` specifies the default receiver if no routes match.
- A `routes:` list allows you to define specific matching rules based on alert labels (like `severity`, `app_type`, etc.).
- Each route can specify its own `receiver` and can have sub-routes for more detailed label matching.

Go through alertmanager.yml
In this structure:
- All alerts first check if `app_type` matches "linux" or "windows"; they go to `ss-admin`.
- Then, based on `app_type` label, alerts get routed to either the `linux-admin` or `windows-admin` receiver.
- Within that, there's further routing by `severity`: "critical" alerts are sent to the manager, and "warning" alerts to the lead.

This allows fine control over how alerts are delivered based on their meaning and scope, keeping teams focused and notifications relevant.

###### BlackBox Exporter 
link to package: https://github.com/prometheus/blackbox_exporter. Runs default at 9115 port
  - Probing exporter used to monitor network endpints like http/s, dns, icmp or tcp
  - This exporter returns detailed metrics about requests
  - Usecase : No knowlegde of sytem internels, respose time, DNS response time to check n/w latency issues, availability of service, n/w health etc
  - /metrics contains the blackbox metrics and /probe contains target metrics

###### Push Gateway
Link: https://github.com/prometheus/pushgateway, runs on port 9091 by default
 - Push gateway is metrics cache for service level batch jobs
  - Used to handle exposition of metrics that have been pushed from short lived jobs like cron or batch.
- API used to push metrics in this gateway:
  ```
  http://<pushgatewayhost>:<port>/metrics/job/<job_name>/[<label_name>/<label_value>]/[<label_name1>/<label_value1>]/..../[<label_name_N>/<label_value_N>]
  ```
- Link for python pushgateway: https://github.com/prometheus/client_python#exporting-to-a-pushgateway
  - Drawbacks: Availability, Manual Deletion, auto instance health monitoring is via up metric
- You can push using the command below:
  ```bash
  echo "demo_metric 123" | curl --data-binary @- http://localhost:9091/metrics/job/demo_pg_job/instance/demo_instance/event/add
  ```

###### Grafana
 - For external dashboards
 - Link: https://grafana.com/grafana/download, default port 3000
 - Add Prometheus as its data source













