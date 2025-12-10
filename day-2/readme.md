
# Prometheus Monitoring - Complete Guide

A comprehensive, beginner-friendly guide to understand Prometheus, metrics, monitoring, and how to set it up on Kubernetes.

---

## Table of Contents
1. [Metrics vs Monitoring](#metrics-vs-monitoring)
2. [What is Prometheus?](#what-is-prometheus)
3. [Why Prometheus?](#why-prometheus)
4. [Prometheus Architecture](#prometheus-architecture)
5. [Key Components Explained](#key-components-explained)
6. [How Prometheus Works](#how-prometheus-works)
7. [Metric Types](#metric-types)
8. [PromQL Basics](#promql-basics)
9. [Installation on Kubernetes](#installation-on-kubernetes)
10. [Verification and Access](#verification-and-access)
11. [Cleanup](#cleanup)

---

## Metrics vs Monitoring

### What are Metrics?[126]

**Metrics** are **measurements or data points** that tell you what is happening in your system.

#### Simple Definition

Think of metrics like **vital signs for your system**:
- Your body has: heart rate, blood pressure, temperature
- Your system has: CPU usage, memory usage, request count

#### Examples of Metrics

```
Number of steps you walk daily: 8,500 steps
Your heart rate: 72 beats per minute
Outside temperature: 25°C
CPU usage: 45%
Memory usage: 6.2GB out of 8GB
Response time: 250ms
Error count: 5 errors in last hour
```

#### Characteristics of Metrics

- **Quantitative** - Always numbers, not descriptions
- **Point-in-time** - Captured at specific moments
- **Lightweight** - Easy to store and transmit
- **Comparable** - Can be compared over time

### What is Monitoring?[126]

**Monitoring** is the **process of keeping an eye on metrics over time** to understand what's normal, identify changes, and detect problems.

#### Simple Definition

Think of monitoring like **watching your fitness progress**:
- You measure steps daily
- You check heart rate regularly
- You see if you're meeting fitness goals
- You notice if something changes

#### Monitoring in Action

```
Monday: CPU = 45%
Tuesday: CPU = 48%
Wednesday: CPU = 52%
Thursday: CPU = 65%
Friday: CPU = 82% ← Alert! Something changed!
```

**Monitoring is the action of repeatedly checking metrics and looking for patterns or problems.**

### Key Difference[126]

| Aspect | Metrics | Monitoring |
|--------|---------|-----------|
| **What it is** | Data points (numbers) | Process of watching data |
| **Example** | "CPU is at 45%" | "CPU increased from 45% to 82% in 4 days" |
| **Time** | Single moment | Over time (trends) |
| **Purpose** | Raw data | Understanding and alerting |

#### Real Analogy

- **Metrics** = Taking your temperature once (37°C)
- **Monitoring** = Checking temperature daily to see if you're getting sick

---

## What is Prometheus?[126][128]

**Prometheus** is an **open-source systems monitoring and alerting toolkit** originally built at SoundCloud.

### Simple Definition

Prometheus is a **health monitoring system** that:
1. Watches your systems (applications, servers, databases)
2. Collects metrics from them
3. Stores the data for analysis
4. Helps you find problems
5. Sends alerts when something goes wrong

### Why "Prometheus"?[126]

In Greek mythology, Prometheus was a Titan who brought fire (knowledge/light) to humanity. Similarly, Prometheus brings visibility to your systems!

### Key Characteristics[126][128]

- **Open Source** - Free to use and modify
- **Pull-based** - Prometheus pulls data from targets (vs push)
- **Time-Series Database** - Stores data with timestamps
- **Powerful Query Language** - PromQL for analysis
- **Alerting** - Send alerts when thresholds are exceeded
- **Kubernetes-Native** - Built for cloud-native environments
- **Simple Setup** - Easy to install and configure

### Real-World Usage[126]

Companies using Prometheus:
- Netflix (massive scale monitoring)
- Uber (critical infrastructure)
- Airbnb (service monitoring)
- Google Cloud (recommended solution)
- Kubernetes (built-in support)

---

## Why Prometheus?[126][131]

### Problems Prometheus Solves

#### Problem 1: Too Many Systems to Monitor

```
Without Prometheus:
- 50 servers: Monitor each manually? Impossible
- 1000 microservices: Check each one? Takes hours
- Dynamic Kubernetes: Pods come and go, can't track manually

With Prometheus:
- Automatically discovers all services
- Monitors everything in one place
- Scales with your infrastructure
```

#### Problem 2: Finding Problems is Slow

```
Without Prometheus:
User: "App is slow!"
You: Start checking individual servers... 30 minutes later... finally found problem

With Prometheus:
User: "App is slow!"
You: Look at Prometheus dashboard... found problem in 2 minutes
```

#### Problem 3: Data is Lost

```
Without Prometheus:
Log is created → Pod crashes → Log deleted → No way to see what happened

With Prometheus:
Metrics collected → Stored for weeks → Can analyze historical data
```

### Key Advantages[126][131]

1. **Real-time Monitoring** - See what's happening now
2. **Historical Data** - Analyze past performance
3. **Alert Capability** - Notify teams automatically
4. **Easy to Scale** - Works with 10 or 10,000 metrics
5. **Built for Kubernetes** - Native service discovery
6. **Powerful Queries** - Find exactly what you need
7. **Flexible** - Monitor anything with an exporter

---

## Prometheus Architecture

### High-Level Overview[126][128]

```
┌─────────────────────────────────────────────────────┐
│           Your Applications & Infrastructure         │
│  (Servers, Databases, Microservices, Containers)   │
│                    ↓ Expose metrics                 │
├─────────────────────────────────────────────────────┤
│  Prometheus Server (Active Scraper)                 │
│  ├─ Retrieval (Pull metrics at regular intervals)   │
│  ├─ TSDB (Time Series Database - Storage)           │
│  └─ HTTP Server (Query API)                         │
│                    ↓                                 │
├─────────────────────────────────────────────────────┤
│  Analysis & Alerting                                │
│  ├─ PromQL (Query language)                         │
│  ├─ Alertmanager (Send alerts)                      │
│  ├─ Grafana (Visualize)                             │
│  └─ API Clients (Custom apps)                       │
└─────────────────────────────────────────────────────┘
```

### Architecture Components
![Prometheus Architecture](images/prometheus-architecture.gif)

---

## Key Components Explained

### Component 1: Service Discovery[128][131]

**What it does:**
Automatically finds what needs to be monitored.

**Think of it as:**
A detective that discovers all your services without you telling it.

#### How Service Discovery Works[128]

**Problem it solves:**
```
Old way: Manually list 1000 services in configuration
New way: Automatically discover services as they start/stop
```

#### Service Discovery Types[128]

1. **Kubernetes Discovery**
   - Auto-discovers pods, services, nodes
   - Best for Kubernetes environments
   - Uses Kubernetes API

2. **File Discovery**
   - Read targets from configuration files
   - Best for static environments
   - Manual updates needed

3. **AWS Discovery**
   - Discovers EC2 instances
   - Discovers RDS databases
   - Dynamic on cloud

4. **Consul Discovery**
   - Works with Consul service registry
   - Dynamic service discovery

**Real Example:**
```
Kubernetes Service Discovery:
Prometheus queries Kubernetes API every 30 seconds
Kubernetes responds: "Here are 50 pods running today"
Prometheus automatically scrapes all 50 pods
A pod crashes → New pod created
Prometheus automatically scrapes new pod (no manual update!)
```

### Component 2: Prometheus Server - Retrieval Module[126]

**What it does:**
Actively "pulls" metrics from targets at regular intervals.

**Think of it as:**
A postal worker that visits mailboxes (targets) on schedule to collect mail (metrics).

#### How it Works[126]

```
Every 15 seconds (default):
Prometheus → Scrapes target endpoint
           → Reads metrics in Prometheus format
           → Stores in TSDB
           → Repeat for all targets
```

**Pull vs Push Model:**

```
Pull (Prometheus way):
Prometheus: "Hey app, what metrics do you have?"
App: "CPU=45%, Memory=2GB"
Prometheus: Stores data

Push (Other systems):
App: "Hey, my metrics are CPU=45%, Memory=2GB"
Monitor: "Thanks, storing..."

Prometheus uses PULL - more reliable!
```

### Component 3: TSDB (Time Series Database)[126]

**What it does:**
Stores metrics efficiently with timestamps.

**Think of it as:**
A library that stores weather records perfectly organized.

#### How Data is Stored[126]

```
Example metric stored:

Metric Name: http_requests_total
Labels: {job="checkout", instance="server-1", method="POST", status="200"}
Timestamps: 
  10:00:00 → value: 1000
  10:00:15 → value: 1015
  10:00:30 → value: 1031
  10:00:45 → value: 1048
```

**Why TSDB?**
- Optimized for time-series data
- Efficient queries over time ranges
- Data compressed on disk
- Default retention: 15 days (configurable)

### Component 4: Alertmanager[126]

**What it does:**
Manages and sends alerts when rules are triggered.

**Think of it as:**
A secretary that groups alerts and sends them to the right people.

#### Alertmanager Features[126]

1. **Deduplication** - Don't send 1000 copies of same alert
2. **Grouping** - Group related alerts together
3. **Routing** - Send to right team/channel
4. **Throttling** - Don't spam with repeated alerts

#### Alert Flow[126]

```
1. Prometheus evaluates alert rule
   "If error_rate > 5% for 5 minutes"
   
2. Condition met → Alert triggered
   
3. Alertmanager receives alert
   
4. Alertmanager groups/deduplicates
   
5. Routes to destinations:
   - PagerDuty (page on-call engineer)
   - Slack (notify team)
   - Email (formal notification)
   - Custom webhook (your system)
```

### Component 5: Exporters[126]

**What it does:**
Collect metrics from systems and expose them in Prometheus format.

**Think of it as:**
A translator that converts system data to Prometheus format.

#### Common Exporters[126]

1. **Node Exporter**
   - CPU, memory, disk, network metrics
   - For Linux/Unix systems
   - Runs on each server

2. **MySQL Exporter**
   - Database performance metrics
   - Query count, connections, performance

3. **PostgreSQL Exporter**
   - Database metrics for PostgreSQL

4. **Redis Exporter**
   - Cache performance metrics

5. **Application Exporters**
   - Custom application metrics
   - Business metrics

#### How Exporters Work[126]

```
Server metrics (CPU, Memory, Disk)
         ↓
Node Exporter (listens on :9100)
         ↓ (converts to Prometheus format)
/metrics endpoint
         ↓
Prometheus scrapes /metrics
         ↓
Stores in TSDB
```

### Component 6: Prometheus Web UI[126]

**What it does:**
Provides web interface to query and visualize metrics.

**Features:**
- Run PromQL queries
- View graphs
- Check scrape targets
- View alert rules
- Accessible at `http://localhost:9090`

### Component 7: Grafana[126]

**What it does:**
Beautiful dashboards to visualize Prometheus metrics.

**Features:**
- Rich visualization options (100+ types)
- Custom dashboards
- Alert panels
- Share dashboards with team
- Multiple data source support

---

## How Prometheus Works

### Step-by-Step Process[126][128]

```
1. SERVICE DISCOVERY
   Prometheus queries service registry (Kubernetes API, files, AWS, etc.)
   ↓
2. BUILD TARGET LIST
   Gets list of all endpoints to monitor
   ↓
3. SCRAPE METRICS
   Every 15-30 seconds:
   - HTTP GET request to /metrics endpoint
   - Reads metric data
   ↓
4. PARSE METRICS
   Parses Prometheus format:
   metric_name{labels} value timestamp
   ↓
5. STORE IN TSDB
   Stores time-series data with:
   - Metric name
   - Label values
   - Timestamp
   - Value
   ↓
6. EVALUATE ALERT RULES
   Checks if any alert conditions met
   ↓
7. TRIGGER ALERTS (IF NEEDED)
   Sends to Alertmanager
   ↓
8. QUERY & ANALYZE
   Users query with PromQL
```

### Example: Scraping Process[126]

```
Prometheus Configuration:
scrape_configs:
  - job_name: 'checkout-service'
    scrape_interval: 15s
    targets: ['checkout:9090']

Execution:
Time 10:00:00 → Scrape checkout:9090/metrics
Response:
  http_requests_total{method="GET",status="200"} 1500
  http_requests_total{method="POST",status="200"} 2500
  process_cpu_seconds_total 45.23

Store in TSDB:
  Time: 10:00:00
  Metric: http_requests_total{method="GET",status="200"}
  Value: 1500

Time 10:00:15 → Scrape again (repeat)
```

---

## Metric Types

Prometheus supports 4 types of metrics:[132]

### Type 1: Counter[132]

**What it is:**
A metric that only increases (or resets to zero).

**Characteristics:**
- Cumulative (always goes up)
- Never decreases
- Can reset to zero only
- Best for counting events

**Use Cases:**
- Total requests received
- Total errors
- Tasks completed
- Bytes transferred

**Example:**

```
Metric: http_requests_total
Value at 10:00 = 1000 requests
Value at 10:01 = 1050 requests (increased by 50)
Value at 10:02 = 1100 requests (increased by 50)

If it resets to 0, it means system restarted
```

**PromQL Query:**

```
# See how many requests in the last 5 minutes
rate(http_requests_total[5m])

# Result might be: 10 requests per second
```

### Type 2: Gauge[132]

**What it is:**
A metric that can go up or down.

**Characteristics:**
- Can increase or decrease
- Represents current state
- Like a thermometer reading
- Best for measurements

**Use Cases:**
- CPU usage (goes up and down)
- Memory usage
- Temperature
- Current connections
- Queue length

**Example:**

```
Metric: node_memory_MemAvailable_bytes
Value at 10:00 = 4GB available
Value at 10:01 = 2GB available (went down)
Value at 10:02 = 3.5GB available (went up)
```

**PromQL Query:**

```
# Current memory available
node_memory_MemAvailable_bytes

# Memory usage percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

### Type 3: Histogram[132]

**What it is:**
Tracks distribution of values (like how many requests took 10ms vs 100ms vs 1s).

**Characteristics:**
- Records observations
- Buckets observations into ranges
- Good for latency tracking
- Gives count + sum of values

**Use Cases:**
- Request duration distribution
- Response time buckets
- Latency analysis
- SLA tracking

**Example:**

```
Metric: http_request_duration_seconds

Bucket 0.1s: 100 requests (took less than 0.1s)
Bucket 0.5s: 500 requests (took 0.1-0.5s)
Bucket 1.0s: 800 requests (took 0.5-1.0s)
Bucket 5.0s: 900 requests (took 1.0-5.0s)
Bucket +Inf: 900 requests (took more than 5s)

Sum: 3100 requests total
```

**PromQL Query:**

```
# Average request duration
rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m])

# 95th percentile latency
histogram_quantile(0.95, http_request_duration_seconds_bucket)
```

### Type 4: Summary[132]

**What it is:**
Similar to histogram but records quantiles directly.

**Characteristics:**
- Pre-computed percentiles
- Sum and count included
- Fewer storage needs than histogram
- Good for client-side percentiles

**Use Cases:**
- Request duration with percentiles
- Response sizes
- Processing time

**Example:**

```
Metric: http_request_duration_seconds

50th percentile (median): 150ms
95th percentile: 800ms
99th percentile: 2000ms

Count: 10,000 requests
Sum: 1,500,000 ms total
```

### Comparison of Metric Types

| Type | Use Case | Can Decrease | Example |
|------|----------|--------------|---------|
| Counter | Total events | ❌ (only reset) | Total requests |
| Gauge | Current state | ✅ Yes | CPU usage |
| Histogram | Distribution | ❌ | Request latency |
| Summary | Percentiles | ❌ | Request duration |

---

## PromQL Basics

### What is PromQL?[127][130]

**PromQL** (Prometheus Query Language) is the **query language** to retrieve and analyze metrics from Prometheus.

**Simple Definition:**
PromQL is like SQL but for time-series metrics. It helps you ask questions about your metrics.

### Basic PromQL Syntax[127][130]

```
metric_name{label="value"}[time_range]
```

#### Components:

1. **Metric Name** - What metric to query
   - `http_requests_total`
   - `node_cpu_seconds_total`

2. **Label Selectors** - Filter by labels
   - `{job="api"}` - Filter by job label
   - `{method="GET", status="200"}` - Multiple filters

3. **Time Range** - Historical data
   - `[5m]` - Last 5 minutes
   - `[1h]` - Last 1 hour
   - `[7d]` - Last 7 days

### PromQL Query Examples[127][130]

#### Example 1: Simple Metric Query

```promql
# Get current value of metric
http_requests_total

# Returns: All http_requests_total metrics with current value
```

#### Example 2: Filter by Labels

```promql
# Only GET requests
http_requests_total{method="GET"}

# GET requests with status 200
http_requests_total{method="GET", status="200"}

# NOT equal (!=)
http_requests_total{status!="200"}
```

#### Example 3: Time Range Queries

```promql
# Average over 5 minutes
rate(http_requests_total[5m])
# Shows requests per second over last 5 min

# Sum over 1 hour
increase(http_requests_total[1h])
# Shows total increase over last hour
```

#### Example 4: Comparison Operators[127]

```promql
# Greater than 80%
node_cpu_usage > 80

# Less than 1000
http_requests_total < 1000

# Equal to value
disk_free == 50GB
```

#### Example 5: Aggregation Functions[127]

```promql
# Sum all metrics
sum(http_requests_total)

# Average across instances
avg(node_memory_usage)

# Maximum value
max(http_request_duration_seconds)

# Group by label
sum by (job) (http_requests_total)
# Shows total per job
```

#### Example 6: Rate of Change[127]

```promql
# Requests per second over last 5 minutes
rate(http_requests_total[5m])

# Increase in requests over last hour
increase(http_requests_total[1h])
```

#### Example 7: Complex Query - CPU Usage[130]

```promql
# CPU usage percentage (excluding idle time)
sum by (cpu) (rate(node_cpu_seconds_total{mode!="idle"}[5m])) * 100
```

### Common PromQL Functions[127][130]

| Function | Purpose | Example |
|----------|---------|---------|
| `rate()` | Rate of change per second | `rate(requests[5m])` |
| `increase()` | Total increase over time | `increase(errors[1h])` |
| `sum()` | Sum all values | `sum(memory)` |
| `avg()` | Average value | `avg(latency)` |
| `max()` | Maximum value | `max(cpu)` |
| `min()` | Minimum value | `min(memory)` |
| `count()` | Count metrics | `count(up)` |
| `histogram_quantile()` | Percentile | `histogram_quantile(0.95, latency)` |

### Helpful PromQL Tips[127][130]

```promql
# View first few results
http_requests_total limit 5

# Skip first N results  
http_requests_total offset 10

# Combine conditions with 'and'
cpu_usage > 80 and memory_usage > 90

# Combine conditions with 'or'
cpu_usage > 90 or memory_usage > 95
```

---

## Installation on Kubernetes

### Prerequisites[126][131]

Before installing Prometheus:
- ✅ AWS EKS Cluster (or any Kubernetes)
- ✅ kubectl installed and configured
- ✅ Helm package manager installed
- ✅ AWS CLI configured
- ✅ eksctl installed

### Step 1: Create EKS Cluster

**Create cluster without nodegroup (we'll add it separately):**

```bash
eksctl create cluster --name=observability \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup
```

**What this does:**
- Creates EKS cluster named "observability"
- In US East region (Virginia)
- Across 2 availability zones (for high availability)
- Without worker nodes (we'll add them next)

**Wait for cluster to be created (5-10 minutes)**

### Step 2: Associate IAM OIDC Provider

**Why this is needed:**
AWS IAM integration for service accounts. This lets Kubernetes pods authenticate with AWS.

```bash
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster observability \
    --approve
```

**What this does:**
- Links Kubernetes service accounts to AWS IAM roles
- Allows pods to have AWS permissions

### Step 3: Create Node Group

**Add worker nodes to the cluster:**

```bash
eksctl create nodegroup --cluster=observability \
                        --region=us-east-1 \
                        --name=observability-ng-private \
                        --node-type=t3.medium \
                        --nodes-min=2 \
                        --nodes-max=3 \
                        --node-volume-size=20 \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access \
                        --node-private-networking
```

**What each parameter means:**

- `--node-type=t3.medium` - Instance type (small, cost-effective)
- `--nodes-min=2` - Minimum 2 nodes (HA)
- `--nodes-max=3` - Maximum 3 nodes (auto-scaling)
- `--node-volume-size=20` - 20GB storage per node
- `--managed` - Use managed nodegroups
- Various `--*-access` flags - Permissions for different services
- `--node-private-networking` - Nodes are private (not internet-facing)

**Wait for nodegroup creation (5-10 minutes)**

### Step 4: Configure kubectl

**Update kubeconfig to connect to cluster:**

```bash
aws eks update-kubeconfig --name observability
```

**Verify connection:**

```bash
kubectl get nodes
# Should show 2 nodes running
```

### Step 5: Add Prometheus Helm Repository

**Add Prometheus community charts:**

```bash
helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts

helm repo update
```

**What this does:**
- Adds official Prometheus Helm charts repository
- Makes Prometheus installable via Helm

### Step 6: Create Monitoring Namespace

**Create a separate namespace for monitoring:**

```bash
kubectl create ns monitoring
```

**Why separate namespace:**
- Keeps monitoring separate from applications
- Easier to manage permissions
- Easier to delete if needed

### Step 7: Create Custom Values File

**Create `custom_kube_prometheus_stack.yml`:**

```yaml
# custom_kube_prometheus_stack.yml

# Prometheus configuration
prometheus:
  prometheusSpec:
    retention: 7d  # Keep data for 7 days
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: gp2
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 50Gi  # 50GB storage
    
    # Service discovery for Kubernetes
    serviceMonitorSelectorNilUsesHelmValues: false
    
    # Alert evaluation
    evaluationInterval: 15s

# Grafana configuration
grafana:
  enabled: true
  adminPassword: prom-operator  # Default password
  service:
    type: LoadBalancer  # Expose externally

# Alertmanager configuration
alertmanager:
  enabled: true
  alertmanagerSpec:
    storage:
      volumeClaimTemplate:
        spec:
          storageClassName: gp2
          resources:
            requests:
              storage: 10Gi

# Node exporter (collect system metrics)
nodeExporter:
  enabled: true

# Prometheus operator
prometheusOperator:
  enabled: true
```

### Step 8: Install kube-prometheus-stack

**Install Prometheus with Helm:**

```bash
cd day-2

helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f ./custom_kube_prometheus_stack.yml
```

**What this installs:**
- Prometheus Server
- Grafana
- Alertmanager
- Node Exporter
- Prometheus Operator

**Wait for all pods to be ready (3-5 minutes)**

---

## Verification and Access

### Step 1: Verify Installation

**Check all pods are running:**

```bash
kubectl get all -n monitoring
```

**Expected output:**

```
NAME                          READY   STATUS    RESTARTS   AGE
pod/prometheus-0              2/2     Running   0          2m
pod/grafana-xxx               1/1     Running   0          2m
pod/alertmanager-0            1/1     Running   0          2m
pod/node-exporter-xxx         1/1     Running   0          2m
pod/prometheus-operator-xxx   1/1     Running   0          2m
```

**All pods should show READY and Running**

### Step 2: Access Prometheus UI

**Set up port forwarding:**

```bash
kubectl port-forward service/prometheus-operated -n monitoring 9090:9090
```

**If using EC2 instance (open to all interfaces):**

```bash
kubectl port-forward service/prometheus-operated -n monitoring 9090:9090 --address 0.0.0.0
```

**Access Prometheus:**
- Open browser: `http://localhost:9090` (or `http://instance-ip:9090`)
- You should see Prometheus web UI

**What you can do:**
- Query metrics with PromQL
- View graphs
- Check targets (Status → Targets)
- View alert rules (Alerts)

### Step 3: Access Grafana

**Set up port forwarding:**

```bash
kubectl port-forward service/monitoring-grafana -n monitoring 8080:80
```

**If using EC2 instance:**

```bash
kubectl port-forward service/monitoring-grafana -n monitoring 8080:80 --address 0.0.0.0
```

**Access Grafana:**
- Open browser: `http://localhost:8080` (or `http://instance-ip:8080`)
- Login credentials:
  - Username: `admin`
  - Password: `prom-operator` (from values file)

**What you can do:**
- View pre-built dashboards
- Create custom dashboards
- Set up alerting
- Visualize Prometheus metrics

### Step 4: Access Alertmanager

**Set up port forwarding:**

```bash
kubectl port-forward service/alertmanager-operated -n monitoring 9093:9093
```

**If using EC2 instance:**

```bash
kubectl port-forward service/alertmanager-operated -n monitoring 9093:9093 --address 0.0.0.0
```

**Access Alertmanager:**
- Open browser: `http://localhost:9093` (or `http://instance-ip:9093`)

**What you can do:**
- View active alerts
- View alert groups
- Configure notification receivers

### Step 5: Run Sample PromQL Queries

**In Prometheus UI, try these queries:**

```promql
# See which targets are healthy
up

# CPU usage percentage
(1 - avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m]))) * 100

# Memory usage
node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100

# Disk usage
(node_filesystem_size_bytes - node_filesystem_available_bytes) / node_filesystem_size_bytes * 100

# Load average
node_load1
```

---

## Cleanup

### Remove Prometheus Stack

**Uninstall Helm release:**

```bash
helm uninstall monitoring --namespace monitoring
```

**Delete monitoring namespace:**

```bash
kubectl delete ns monitoring
```

### Remove EKS Cluster

**Delete cluster and all resources:**

```bash
eksctl delete cluster --name observability --region us-east-1
```

**This will remove:**
- All nodes
- Load balancers
- VPC
- All other AWS resources

**This takes 10-15 minutes**

---

## Best Practices

### 1. Set Appropriate Retention

```yaml
# Keep data for 7-30 days depending on storage
retention: 7d  # 7 days default
```

### 2. Use Service Monitors

Instead of manual scrape configs, use ServiceMonitor for Kubernetes:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
    interval: 30s
```

### 3. Create Meaningful Dashboards

- System metrics dashboard
- Application performance dashboard
- Business metrics dashboard
- Troubleshooting dashboard

### 4. Set Up Alerts

Define alert rules for:
- High CPU (> 90%)
- High memory (> 85%)
- High error rate (> 5%)
- Service down (no response)

### 5. Regular Backups

Backup Prometheus data regularly for compliance and historical analysis.

### 6. Monitor Prometheus Itself

Monitor:
- Prometheus disk usage
- Prometheus memory usage
- Prometheus scrape latency
- Number of targets scrapped

---

## Common Issues and Solutions

### Issue 1: Prometheus data not showing

**Problem:** No metrics in Prometheus

**Check:**
```bash
# Check if targets are scraping
kubectl port-forward service/prometheus-operated 9090:9090
# Then go to Status → Targets
```

**Solution:**
- Verify targets are in "Up" state
- Check if applications are exposing /metrics endpoint

### Issue 2: Out of disk space

**Problem:** Prometheus fills up disk

**Solution:**
- Reduce retention: `retention: 3d`
- Increase storage: Edit PVC
- Delete old data: Use retention policy

### Issue 3: High memory usage

**Problem:** Prometheus consuming too much RAM

**Solution:**
```yaml
resources:
  limits:
    memory: "2Gi"
  requests:
    memory: "1Gi"
```

### Issue 4: Metrics missing from applications

**Problem:** Application metrics not showing up

**Check:**
1. Is app exposing /metrics endpoint?
2. Is ServiceMonitor configured?
3. Are labels correct?

---

## Conclusion

Prometheus provides powerful, flexible monitoring for Kubernetes:

✅ Automatic service discovery  
✅ Scalable metric collection  
✅ Powerful query language (PromQL)  
✅ Built-in alerting  
✅ Integration with Grafana  
✅ Industry standard  

With Prometheus and Grafana, you have enterprise-grade monitoring at your fingertips!

---

## Additional Resources

- [Prometheus Official Docs](https://prometheus.io/docs/)
- [PromQL Documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/)
- [Prometheus Kubernetes Guide](https://prometheus.io/docs/prometheus/latest/installation/)
- [Grafana Documentation](https://grafana.com/docs/grafana/)
- [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
