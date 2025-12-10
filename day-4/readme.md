
# Instrumentation & Custom Metrics - Complete Guide

A comprehensive, beginner-friendly guide to understanding instrumentation, custom metrics, and how to implement monitoring in your applications.

---

## Table of Contents
1. [What is Instrumentation?](#what-is-instrumentation)
2. [Why Instrumentation Matters](#why-instrumentation-matters)
3. [How Instrumentation Works](#how-instrumentation-works)
4. [Types of Metrics](#types-of-metrics)
5. [Instrumentation in Prometheus](#instrumentation-in-prometheus)
6. [Custom Metrics in Node.js](#custom-metrics-in-nodejs)
7. [Alertmanager Configuration](#alertmanager-configuration)
8. [Complete Implementation Guide](#complete-implementation-guide)
9. [Testing & Validation](#testing--validation)
10. [Best Practices](#best-practices)

---

## What is Instrumentation?

### Definition

**Instrumentation** is the process of **adding monitoring capabilities** to your applications, systems, or services by embedding code or using tools to collect metrics, logs, or traces.

### Simple Definition

Think of instrumentation like **adding sensors to your car**:
- Without sensors: You don't know how much fuel is left, engine temperature, battery status
- With sensors: Dashboard shows everything - fuel, temperature, battery, speed

In applications:
- Without instrumentation: You don't know how many requests, errors, or response time
- With instrumentation: You know exactly what's happening inside your app

### Real-World Analogy

```
Medical Doctor checking patient health:

Without Instrumentation:
❌ "How is the patient?" 
❌ Just looking at them, no data

With Instrumentation:
✅ Checking heart rate (counter - increases)
✅ Checking blood pressure (gauge - up/down)
✅ Checking temperature (gauge - fluctuates)
✅ Checking test results (histogram - distribution)
✅ Complete picture of health!
```

### Key Concept

**Instrumentation = Adding visibility into your system**

---

## Why Instrumentation Matters

### 1. Visibility into Application State

**Problem without instrumentation:**
```
User: "App is slow!"
You: "Which endpoint? How slow? How many users affected? No idea!"
```

**With instrumentation:**
```
User: "App is slow!"
You: Check metrics → "/api/checkout is slow - 2000ms"
     Check traces → "Database query taking 1800ms"
     Check logs → "Database connection timeout"
Root cause found in seconds!
```

### 2. Metrics Collection for Performance

Collect key metrics:
- **Request count** - How many requests per second?
- **Response time** - How fast are we responding?
- **Error rate** - What % of requests fail?
- **Resource usage** - CPU, memory consumption?
- **Business metrics** - Orders per minute, users logged in?

### 3. Troubleshooting & Debugging

When something breaks:
```
❌ Without metrics: Guessing, trial and error, takes hours
✅ With metrics: Identify exact problem in minutes
```

### 4. Proactive Alerting

```
Without: Problem happens → Users complain → You investigate
With: Metric crosses threshold → Alert fires → You fix before users notice
```

### 5. Performance Optimization

```
With instrumentation, you can:
- Find slowest API endpoints
- Identify memory leaks
- Detect performance regressions after deployments
- Plan capacity based on trends
```

### 6. Compliance & Auditing

Track:
- API usage
- Authentication events
- Data access patterns
- Performance SLAs

---

## How Instrumentation Works

### 3-Step Process

```
Step 1: ADD CODE
├─ Add monitoring code to application
├─ Define what metrics to collect
└─ Define how often to collect

Step 2: COLLECT DATA
├─ Application runs normally
├─ Metrics are generated continuously
└─ Data accumulated in memory/file

Step 3: EXPOSE & SCRAPE
├─ Expose metrics at /metrics endpoint
├─ Prometheus scrapes the endpoint
├─ Data stored in TSDB
└─ Now queryable and analyzable
```

### Example: Instrumenting an HTTP Server

```javascript
// Step 1: Import monitoring library
const client = require('prom-client');

// Step 2: Define what to measure
const httpRequestCounter = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'status', 'endpoint']
});

// Step 3: Measure when requests happen
app.use((req, res, next) => {
  res.on('finish', () => {
    httpRequestCounter.inc({
      method: req.method,
      status: res.statusCode,
      endpoint: req.path
    });
  });
  next();
});

// Step 4: Expose metrics
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(await client.register.metrics());
});
```

**What happens:**
1. Every request increments the counter
2. `/metrics` endpoint returns all metrics
3. Prometheus scrapes `/metrics` every 15 seconds
4. You can now query `http_requests_total` in Prometheus

### Instrumentation Types

**1. Code-Level Instrumentation**
```
Add monitoring code directly in application
✅ Most accurate metrics
✅ Custom business metrics
❌ Requires code changes
```

**2. Library/Agent-Based**
```
Use libraries like prom-client, OpenTelemetry
✅ Less code required
✅ Standardized approach
❌ Limited customization
```

**3. Exporters**
```
Use dedicated tools to collect from systems
✅ No application changes
✅ Good for legacy systems
❌ Limited detail
```

---

## Types of Metrics

### Type 1: Counter

**What it is:**
A metric that **only increases** (never decreases).

**Characteristics:**
- Cumulative (always goes up)
- Can reset to zero only when system restarts
- Starts at 0
- Useful for counting events

**Real Examples:**
```
http_requests_total: 1000 → 1001 → 1002 → 1003
                     (went up by 1 each request)

Container restarts: 0 → 1 → 2 → 3
                   (went up each time it crashed)

Errors total: 50 → 51 → 52
             (cumulative errors)
```

**Use Cases:**
- Total HTTP requests received
- Total errors encountered
- Total tasks completed
- Total bytes sent/received
- Total database queries

**PromQL Example:**
```promql
# How many requests per second in last 5 min?
rate(http_requests_total[5m])

# Total increase in errors over last hour?
increase(errors_total[1h])
```

**Key Insight:**
To get useful info from a counter, use `rate()` or `increase()` functions!

---

### Type 2: Gauge

**What it is:**
A metric that **can go up or down**.

**Characteristics:**
- Current value (snapshot)
- Can increase or decrease
- Represents "state" at a moment
- Like a thermometer reading

**Real Examples:**
```
CPU Usage: 45% → 48% → 52% → 38% → 45%
          (up and down)

Memory Available: 4GB → 3.5GB → 2GB → 3GB
                 (fluctuates)

Active Connections: 100 → 120 → 95 → 110
                   (goes up when connecting, down when disconnecting)
```

**Use Cases:**
- CPU usage percentage
- Memory usage
- Disk space available
- Current temperature
- Number of active users/connections
- Queue length

**PromQL Example:**
```promql
# Current CPU usage
node_cpu_usage_percent

# Memory available
node_memory_MemAvailable_bytes

# Which services have high memory?
topk(5, container_memory_usage_bytes)
```

**Key Insight:**
Gauges show current state, not cumulative!

---

### Type 3: Histogram

**What it is:**
A metric that **tracks distribution** of values in buckets.

**Characteristics:**
- Samples observations (like latency, size)
- Groups them into buckets/ranges
- Provides count and sum
- Good for measuring latencies

**How It Works:**

```
Request Duration Histogram:
(tracking how long requests take)

Bucket 0.1s:  ████ (100 requests took < 0.1s)
Bucket 0.5s:  ███████████████ (500 requests took 0.1-0.5s)
Bucket 1.0s:  ██████████████████ (800 requests took 0.5-1.0s)
Bucket 5.0s:  ████████ (100 requests took 1-5s)
Bucket +Inf:  ███ (20 requests took > 5s)

Total: 1520 requests
Sum: 850 seconds total
```

**Real Examples:**
```
API Response Times:
- 95% of requests: < 500ms ✅
- 5% of requests: > 500ms ⚠️

Database Query Duration:
- 90% of queries: < 50ms ✅
- 10% of queries: 50-200ms ⚠️

File Upload Size:
- Most: < 10MB
- Some: 10-100MB
- Few: > 100MB
```

**Use Cases:**
- Request/response latency
- Query duration
- Message size
- Processing time
- Any performance metric

**PromQL Example:**
```promql
# 95th percentile latency (95% of requests faster than this)
histogram_quantile(0.95, http_request_duration_seconds_bucket)

# Average request duration
rate(http_request_duration_seconds_sum[5m]) / 
rate(http_request_duration_seconds_count[5m])
```

**Key Insight:**
Histograms let you understand distribution, not just average!

---

### Type 4: Summary

**What it is:**
Similar to histogram, but with **pre-calculated percentiles**.

**Characteristics:**
- Tracks observations like histogram
- Pre-computes percentiles
- Less storage than histogram
- Good for client-side percentile calculation

**Real Examples:**
```
Request Duration Summary:
50th percentile (median): 150ms
90th percentile: 500ms
95th percentile: 850ms
99th percentile: 2000ms

Summary for processing time:
50th: 10ms
95th: 45ms
99th: 100ms
```

**Use Cases:**
- Request latency with percentiles
- Processing time tracking
- Any metric needing percentiles

**PromQL Example:**
```promql
# Get 95th percentile directly
http_request_duration_summary_seconds{quantile="0.95"}
```

---

## Instrumentation in Prometheus

### How Prometheus Gets Metrics

```
Your Application
    ↓ (exposes)
/metrics Endpoint
    ↓ (Prometheus pulls every 15 seconds)
Prometheus Server
    ↓ (stores in TSDB)
Time-Series Database
    ↓ (available for queries)
Prometheus UI / Grafana
```

### Exporters: The Translation Layer

**Problem:** Many systems don't natively expose Prometheus metrics.

**Solution:** Use exporters!

```
System (MySQL, Linux, Redis)
    ↓
Exporter (translates to Prometheus format)
    ↓
/metrics endpoint
    ↓
Prometheus scrapes
```

### Common Exporters

**1. Node Exporter**
```
Exports: System-level metrics from Linux/Unix servers
Metrics: CPU, memory, disk, network, processes
Usage: Runs on each server you want to monitor
Example metrics:
  - node_cpu_seconds_total
  - node_memory_MemAvailable_bytes
  - node_filesystem_size_bytes
```

**2. MySQL Exporter**
```
Exports: Database metrics from MySQL
Metrics: Queries per second, connections, replication lag
Usage: Runs alongside MySQL
Example metrics:
  - mysql_global_status_questions
  - mysql_global_status_threads_connected
  - mysql_slave_status_seconds_behind_master
```

**3. PostgreSQL Exporter**
```
Exports: Database metrics from PostgreSQL
Metrics: Query duration, table size, cache hits
Usage: Runs alongside PostgreSQL
Example metrics:
  - pg_stat_statements_query_time_seconds
  - pg_table_total_size_bytes
```

**4. Custom Exporters**
```
Exports: Metrics from your custom applications
Usage: Write in Node.js, Python, Go, Java, etc.
Libraries: prom-client (Node.js), prometheus_client (Python)
```

### Custom Metrics in Your Application

**Why custom metrics?**
```
Generic metrics tell you:
- "CPU is at 80%" (infrastructure level)

Custom metrics tell you:
- "Users purchased 500 items" (business level)
- "/api/checkout took 2 seconds" (application level)
- "Failed to connect to payment gateway" (service level)

Generic + Custom = Complete observability!
```

---

## Custom Metrics in Node.js

### Using prom-client Library

The `prom-client` library makes it easy to instrument Node.js applications.

### Installation

```bash
npm install prom-client express
```

### 4 Metric Types Implementation

#### 1. Counter Example

```javascript
const client = require('prom-client');

// Define counter
const httpRequestsTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'status', 'endpoint']
});

// Use in application
app.get('/api/users', (req, res) => {
  try {
    // ... handle request
    res.json({users: []});
    
    // Increment counter
    httpRequestsTotal.inc({
      method: 'GET',
      status: 200,
      endpoint: '/api/users'
    });
  } catch (error) {
    res.status(500).json({error});
    httpRequestsTotal.inc({
      method: 'GET',
      status: 500,
      endpoint: '/api/users'
    });
  }
});
```

**Metric Result:**
```
http_requests_total{method="GET", status="200", endpoint="/api/users"} 1000
http_requests_total{method="GET", status="500", endpoint="/api/users"} 5
```

#### 2. Gauge Example

```javascript
const memoryUsageGauge = new client.Gauge({
  name: 'node_memory_usage_bytes',
  help: 'Memory usage in bytes'
});

// Update every 10 seconds
setInterval(() => {
  const memUsage = process.memoryUsage().heapUsed;
  memoryUsageGauge.set(memUsage);
}, 10000);
```

**Metric Result:**
```
node_memory_usage_bytes 536870912  (changes over time)
```

#### 3. Histogram Example

```javascript
const requestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'endpoint'],
  buckets: [0.1, 0.5, 1, 2, 5]  // 100ms, 500ms, 1s, 2s, 5s
});

// Measure request time
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;  // Convert to seconds
    requestDuration.observe({
      method: req.method,
      endpoint: req.path
    }, duration);
  });
  
  next();
});
```

**Metric Result:**
```
http_request_duration_seconds_bucket{endpoint="/api/users", le="0.1"} 500
http_request_duration_seconds_bucket{endpoint="/api/users", le="0.5"} 950
http_request_duration_seconds_bucket{endpoint="/api/users", le="1"} 1000
http_request_duration_seconds_bucket{endpoint="/api/users", le="+Inf"} 1000
http_request_duration_seconds_sum{endpoint="/api/users"} 750.5
http_request_duration_seconds_count{endpoint="/api/users"} 1000
```

#### 4. Summary Example

```javascript
const requestSummary = new client.Summary({
  name: 'http_request_duration_summary_seconds',
  help: 'HTTP request duration (summary)',
  labelNames: ['method', 'endpoint'],
  percentiles: [0.5, 0.9, 0.95, 0.99]  // Calculate these percentiles
});

// Measure request time
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    requestSummary.observe({
      method: req.method,
      endpoint: req.path
    }, duration);
  });
  
  next();
});
```

**Metric Result:**
```
http_request_duration_summary_seconds{endpoint="/api/users", quantile="0.5"} 0.15
http_request_duration_summary_seconds{endpoint="/api/users", quantile="0.9"} 0.45
http_request_duration_summary_seconds{endpoint="/api/users", quantile="0.95"} 0.65
http_request_duration_summary_seconds{endpoint="/api/users", quantile="0.99"} 0.95
```

### Expose Metrics Endpoint

```javascript
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(await client.register.metrics());
});
```

Now Prometheus can scrape `/metrics` endpoint!

---

## Alertmanager Configuration

### What is Alertmanager?

Alertmanager **manages and routes alerts** based on Prometheus alert rules.

```
Prometheus (detects problem)
    ↓
Triggers alert rule
    ↓
Sends to Alertmanager
    ↓
Alertmanager processes (groups, deduplicates)
    ↓
Routes to destinations (email, Slack, PagerDuty)
    ↓
You get notified!
```

### Alert Rule Example: PodRestart

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: pod-restart-alert
spec:
  groups:
  - name: pod-alerts
    rules:
    - alert: PodRestart
      expr: increase(kube_pod_container_status_restarts_total[1h]) > 2
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "Pod restarted more than 2 times"
        description: "Pod {{ $labels.pod }} restarted {{ $value }} times in last hour"
```

**How it works:**
1. Prometheus evaluates the expression every 15 seconds
2. If a pod restarted > 2 times in last hour: Alert fires
3. Alertmanager receives the alert
4. Sends email notification

### Alertmanager Configuration File

```yaml
global:
  resolve_timeout: 5m
  # SMTP Configuration for email
  smtp_smarthost: smtp.gmail.com:587
  smtp_auth_username: your-email@gmail.com
  smtp_auth_password: your-app-password  # From Google Account settings
  smtp_from: your-email@gmail.com

route:
  receiver: 'default-receiver'
  group_by: ['severity', 'alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h

receivers:
- name: 'default-receiver'
  email_configs:
  - to: your-email@gmail.com
    headers:
      Subject: 'Alert: {{ .GroupLabels.alertname }}'
```

**What each section does:**
- **global**: SMTP settings for email
- **route**: How to route alerts
- **receivers**: Where to send alerts (email, Slack, etc.)

### Getting Gmail Credentials

1. Go to Google Account settings
2. Search for "App passwords"
3. Create new app password for Alertmanager
4. Use the generated password in configuration

---

## Complete Implementation Guide

### Step 1: Application Instrumentation

**File: `application/service-a/index.js`**

```javascript
const express = require('express');
const client = require('prom-client');

const app = express();

// Define metrics
const httpRequestsTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'status', 'endpoint']
});

const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'endpoint'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

const memoryUsageGauge = new client.Gauge({
  name: 'node_memory_usage_bytes',
  help: 'Memory usage'
});

// Middleware to measure request duration
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestsTotal.inc({
      method: req.method,
      status: res.statusCode,
      endpoint: req.path
    });
    httpRequestDuration.observe({
      method: req.method,
      endpoint: req.path
    }, duration);
  });
  
  next();
});

// Update memory gauge every 10 seconds
setInterval(() => {
  memoryUsageGauge.set(process.memoryUsage().heapUsed);
}, 10000);

// Routes
app.get('/', (req, res) => {
  res.json({status: 'Running'});
});

app.get('/healthy', (req, res) => {
  res.json({status: 'Healthy'});
});

app.get('/serverError', (req, res) => {
  res.status(500).json({error: 'Internal Server Error'});
});

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(await client.register.metrics());
});

app.get('/crash', (req, res) => {
  res.json({message: 'Crashing...'});
  setTimeout(() => process.exit(1), 1000);
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Step 2: Dockerize Application

**File: `Dockerfile`**

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package.json .
RUN npm install

COPY index.js .

EXPOSE 3000

CMD ["node", "index.js"]
```

**Build and push:**

```bash
docker build -t your-registry/service-a:v1 .
docker push your-registry/service-a:v1
```

### Step 3: Kubernetes Manifests

**File: `kubernetes-manifest/deployment.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: service-a
  namespace: dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: service-a
  template:
    metadata:
      labels:
        app: service-a
    spec:
      containers:
      - name: service-a
        image: your-registry/service-a:v1
        ports:
        - containerPort: 3000
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
---
apiVersion: v1
kind: Service
metadata:
  name: service-a
  namespace: dev
spec:
  type: LoadBalancer
  selector:
    app: service-a
  ports:
  - port: 80
    targetPort: 3000
```

**Deploy:**

```bash
kubectl create ns dev
kubectl apply -f kubernetes-manifest/
```

### Step 4: Configure Prometheus Service Monitor

**File: `kubernetes-manifest/servicemonitor.yaml`**

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: service-a
  namespace: dev
spec:
  selector:
    matchLabels:
      app: service-a
  endpoints:
  - port: metrics
    interval: 15s
```

### Step 5: Configure Alertmanager

**File: `alerts-alertmanager-servicemonitor-manifest/alertmanager.yaml`**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alertmanager-config
  namespace: monitoring
data:
  alertmanager.yml: |
    global:
      resolve_timeout: 5m
      smtp_smarthost: smtp.gmail.com:587
      smtp_auth_username: your-email@gmail.com
      smtp_auth_password: your-app-password
      smtp_from: your-email@gmail.com
    
    route:
      receiver: 'email-receiver'
      group_by: ['severity']
    
    receivers:
    - name: 'email-receiver'
      email_configs:
      - to: your-email@gmail.com
```

**Apply:**

```bash
kubectl apply -f alerts-alertmanager-servicemonitor-manifest/
```

---

## Testing & Validation

### Step 1: Verify Metrics Collection

```bash
# Get LoadBalancer DNS
kubectl get svc service-a -n dev

# Hit endpoints to generate metrics
curl http://LOAD_BALANCER_DNS/healthy
curl http://LOAD_BALANCER_DNS/serverError
curl http://LOAD_BALANCER_DNS/metrics
```

### Step 2: Check Prometheus Targets

1. Access Prometheus UI: `http://localhost:9090`
2. Go to Status → Targets
3. Verify service-a is showing as "UP"

### Step 3: Query Metrics in Prometheus

```promql
# Total requests
http_requests_total

# Request rate
rate(http_requests_total[5m])

# 95th percentile latency
histogram_quantile(0.95, http_request_duration_seconds_bucket)

# Memory usage
node_memory_usage_bytes
```

### Step 4: Test Alerts

**Trigger PodRestart alert:**

```bash
# Crash pod multiple times
curl http://LOAD_BALANCER_DNS/crash
curl http://LOAD_BALANCER_DNS/crash
curl http://LOAD_BALANCER_DNS/crash

# Wait 1-2 minutes for alert
# Check Prometheus UI: Alerts tab
# Check email for notification
```

### Step 5: Automated Testing Script

**File: `test.sh`**

```bash
#!/bin/bash

LOAD_BALANCER=$1

if [ -z "$LOAD_BALANCER" ]; then
  echo "Usage: ./test.sh <LOAD_BALANCER_DNS>"
  exit 1
fi

while true; do
  # Random endpoint
  ENDPOINTS=("/" "/healthy" "/serverError" "/metrics")
  RANDOM_ENDPOINT=${ENDPOINTS[$RANDOM % ${#ENDPOINTS[@]}]}
  
  echo "Calling: http://$LOAD_BALANCER$RANDOM_ENDPOINT"
  curl -s "http://$LOAD_BALANCER$RANDOM_ENDPOINT" > /dev/null
  
  # Random delay
  sleep $((RANDOM % 3 + 1))
done
```

**Run:**

```bash
chmod +x test.sh
./test.sh your-load-balancer-dns
```

---

## Best Practices

### 1. Choose Right Metric Type

```
Use Counter for:
✅ Total events (requests, errors, completed tasks)

Use Gauge for:
✅ Current state (CPU, memory, temperature)

Use Histogram for:
✅ Distribution (latency, size)

Use Summary for:
✅ Pre-calculated percentiles
```

### 2. Label Cardinality Management

```
❌ BAD - High Cardinality:
{user_id, request_id, session_id}

✅ GOOD - Low Cardinality:
{method, status, endpoint}

Keep label values bounded!
```

### 3. Meaningful Names

```
❌ BAD:
total, count, value

✅ GOOD:
http_requests_total
database_query_duration_seconds
api_error_count
```

### 4. Include Units in Metric Names

```
✅ GOOD:
http_request_duration_seconds
memory_usage_bytes
disk_space_available_gigabytes

❌ BAD:
response_time
memory
disk_space
```

### 5. Set Appropriate Alert Thresholds

```
Test alerts with real data:
- Too strict: False positives (noisy)
- Too loose: Miss real problems

Example:
❌ Alert if response time > 10000ms (too loose)
✅ Alert if response time > 500ms (more reasonable)
```

### 6. Document Your Metrics

```
For each metric, document:
- What it measures
- Why it's important
- Normal range
- Alert thresholds
```

---

## Common Issues & Solutions

### Issue 1: Metrics Not Appearing

**Problem:** No metrics showing in Prometheus

**Check:**
```bash
# Is endpoint responding?
curl http://LOAD_BALANCER/metrics

# Are labels correct?
kubectl describe servicemonitor service-a -n dev

# Check pod logs
kubectl logs -f deployment/service-a -n dev
```

### Issue 2: High Cardinality Warning

**Problem:** Metrics memory growing too fast

**Solution:**
- Remove high-cardinality labels (user_id, request_id)
- Limit label values
- Use drop relabeling to remove unused labels

### Issue 3: Alerts Not Firing

**Problem:** Alert rule not triggering

**Check:**
```promql
# Verify metric exists
kube_pod_container_status_restarts_total

# Check expression
increase(kube_pod_container_status_restarts_total[1h]) > 2
```

### Issue 4: Emails Not Sending

**Problem:** No alert emails received

**Check:**
1. Gmail credentials correct?
2. Less secure apps enabled?
3. Check Alertmanager logs
4. Check SMTP settings

---

## Summary

### 3 Pillars of Instrumentation

1. **Custom Metrics in Applications**
   - Track business metrics
   - Track application performance
   - Use prom-client for Node.js

2. **Alert Rules**
   - Define conditions
   - Set appropriate thresholds
   - Test before production

3. **Alertmanager**
   - Routes alerts
   - Deduplicates and groups
   - Sends notifications

### Implementation Checklist

- ✅ Add prom-client to application
- ✅ Define metrics (counter, gauge, histogram)
- ✅ Expose /metrics endpoint
- ✅ Create ServiceMonitor for Prometheus
- ✅ Create alert rules
- ✅ Configure Alertmanager
- ✅ Test metrics collection
- ✅ Test alert notifications
- ✅ Monitor in production

---

## Next Steps

1. Implement custom metrics in your applications
2. Create dashboards in Grafana for visualization
3. Set up meaningful alerts
4. Test alert notifications
5. Document all metrics and alerts
6. Continuously improve based on patterns

---

## Resources

- [prom-client Documentation](https://github.com/siimon/prom-client)
- [Prometheus Instrumentation](https://prometheus.io/docs/instrumenting/writing_clientlibs/)
- [Alertmanager Configuration](https://prometheus.io/docs/alerting/latest/configuration/)
- [Best Practices](https://prometheus.io/docs/practices/instrumentation/)
