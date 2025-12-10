
# PromQL, Metrics & Labels - Complete Guide

A comprehensive, beginner-friendly guide to understanding Prometheus metrics, labels, and the PromQL query language with practical examples.

---

## Table of Contents
1. [What are Prometheus Metrics?](#what-are-prometheus-metrics)
2. [Understanding Labels](#understanding-labels)
3. [Metrics + Labels Together](#metrics--labels-together)
4. [What is PromQL?](#what-is-promql)
5. [PromQL Basics](#promql-basics)
6. [Selecting Time Series](#selecting-time-series)
7. [Range Vectors](#range-vectors)
8. [Aggregation Operators](#aggregation-operators)
9. [PromQL Functions](#promql-functions)
10. [Real-World Examples](#real-world-examples)
11. [Best Practices](#best-practices)

---

## What are Prometheus Metrics?

### Definition

**Metrics** are the **core data objects** in Prometheus that represent measurements collected from monitored systems.

### Simple Definition

Think of metrics like **vital signs on a medical report**:
- Heart rate: 72 bpm
- Blood pressure: 120/80
- Temperature: 37°C
- Oxygen level: 98%

In Prometheus:
- CPU usage: 45%
- Memory usage: 6.2GB
- Request count: 1000
- Response time: 250ms

### What Metrics Tell You

Metrics provide insights into:
- **Performance** - How fast things are running
- **Health** - Is everything working?
- **Behavior** - What is the system doing?
- **Capacity** - How much resources are being used?

### Characteristics of Metrics

```
✓ Quantitative: Always numbers (45%, 250ms, 1000 requests)
✓ Point-in-time: Captured at specific moments (10:30:45 AM)
✓ Timestamped: When the measurement was taken
✓ Lightweight: Small data points, efficient storage
✓ Comparable: Can be compared over time
✓ Aggregatable: Can be combined with other metrics
```

### Examples of Metrics

```
Metric Name: http_requests_total
Value: 1000 (requests)
Time: 2025-12-10 10:30:45

---

Metric Name: container_cpu_usage_seconds_total
Value: 45.23 (seconds)
Time: 2025-12-10 10:30:45

---

Metric Name: node_memory_MemAvailable_bytes
Value: 4294967296 (bytes = 4GB)
Time: 2025-12-10 10:30:45
```

### Why Metrics Matter

Without metrics:
```
❌ "Is the system working?" → No idea
❌ "How many requests did we handle?" → No idea
❌ "Is memory usage high?" → No idea
❌ "What happened last night?" → No idea
```

With metrics:
```
✅ "Is the system working?" → Check 'up' metric
✅ "How many requests?" → Check 'http_requests_total'
✅ "Memory high?" → Check 'node_memory_usage'
✅ "What happened?" → Check historical metrics
```

---

## Understanding Labels

### What are Labels?

**Labels** are **key-value pairs** attached to metrics that allow you to differentiate between dimensions of a metric.

### Simple Definition

Think of labels like **attributes on a product**:
- **Product:** T-Shirt
- **Attributes:** Size=Medium, Color=Blue, Brand=Nike

In Prometheus:
- **Metric:** `http_requests_total`
- **Attributes:** `method=GET, status=200, endpoint=/api/checkout`

### Why Labels are Important

**Problem without labels:**

```
Metric: http_requests_total = 5000

You ask: "How many were successful vs failed?"
Answer: ?????? No way to know!
```

**Solution with labels:**

```
Metric: http_requests_total{status="200"} = 4950
Metric: http_requests_total{status="500"} = 50

You ask: "How many were successful vs failed?"
Answer: 4950 successful, 50 failed! Perfect!
```

### Common Label Names

```
service: "api-server", "checkout", "payment"
instance: "server-1", "server-2", "server-3"
job: "prometheus", "mysql", "node"
method: "GET", "POST", "PUT", "DELETE"
status: "200", "404", "500"
endpoint: "/api/users", "/api/orders"
namespace: "default", "kube-system", "production"
pod: "api-server-1", "checkout-service-2"
container: "app", "nginx", "mongodb"
environment: "production", "staging", "development"
region: "us-east-1", "eu-west-1", "ap-southeast-1"
```

### Label Characteristics

```
✓ Key-Value Pairs: Always in "key=value" format
✓ Multiple Labels: A metric can have many labels
✓ Dimensional: Add dimensions to metrics
✓ Filterable: Easy to query specific combinations
✓ Queryable: Can aggregate by labels
```

### Label Example

```
Metric Name: http_requests_total
Labels: {
  service="api",
  instance="server-1",
  method="GET",
  status="200",
  endpoint="/api/users"
}
Value: 5000 requests
Time: 2025-12-10 10:30:45
```

### Label Best Practices

#### 1. Meaningful Label Names

```
✅ GOOD:
  - method (clear what it represents)
  - status (obvious: HTTP status)
  - endpoint (which API endpoint)

❌ BAD:
  - m (what does 'm' mean?)
  - x (unclear)
  - val (too generic)
```

#### 2. Consistent Naming Convention

```
✅ Consistent across all metrics:
  - service, instance, method, status
  - Always lowercase
  - Always singular or plural (pick one)

❌ Inconsistent:
  - service, Instance, method, Status (mixed case)
  - service, instance, methods, statuses (inconsistent singular/plural)
```

#### 3. Manage Label Cardinality

**Cardinality** = Number of unique label combinations

```
Example:
Label: user_id
Values: 1 million different users

Problem:
1 million unique combinations = huge storage!
Performance issues!

Solution:
Don't use high-cardinality labels like user_id, session_id, or request_id
Use lower-cardinality labels: method, status, endpoint
```

#### 4. Keep Values Bounded

```
✅ LIMITED VALUES (Low Cardinality):
  - status: "200", "404", "500" (3 values)
  - method: "GET", "POST", "PUT" (3 values)
  - environment: "prod", "staging" (2 values)

❌ UNLIMITED VALUES (High Cardinality):
  - user_id: millions of unique IDs
  - request_id: different every time
  - timestamp: always changing
```

#### 5. Plan Your Label Schema

Define labels upfront:

```
Service Metrics:
- service: service name
- instance: which server
- method: HTTP method
- status: HTTP status
- endpoint: API path

Database Metrics:
- service: service name
- instance: database host
- database: database name
- query_type: SELECT, INSERT, UPDATE

Infrastructure Metrics:
- instance: server
- device: cpu, memory, disk
- mode: user, system, idle
```

---

## Metrics + Labels Together

### Complete Example

```
Metric:  container_cpu_usage_seconds_total
Labels:  {namespace="kube-system", endpoint="https-metrics"}
Value:   45.23
Time:    2025-12-10 10:30:45

Full representation:
container_cpu_usage_seconds_total{namespace="kube-system", endpoint="https-metrics"} 45.23
```

### Real Prometheus Examples

```
Example 1: HTTP Requests
http_requests_total{service="api", method="GET", status="200"} 5000
http_requests_total{service="api", method="GET", status="404"} 15
http_requests_total{service="api", method="POST", status="200"} 2000
http_requests_total{service="api", method="POST", status="500"} 5

Example 2: Pod Memory
container_memory_usage_bytes{pod="api-server-1", namespace="default"} 536870912
container_memory_usage_bytes{pod="api-server-2", namespace="default"} 603979776
container_memory_usage_bytes{pod="checkout-1", namespace="default"} 268435456

Example 3: Node CPU
node_cpu_seconds_total{instance="server-1", mode="user"} 1000
node_cpu_seconds_total{instance="server-1", mode="system"} 500
node_cpu_seconds_total{instance="server-1", mode="idle"} 8500
node_cpu_seconds_total{instance="server-2", mode="user"} 950
```

### How Many Metrics Are Generated?

```
Metric: http_requests_total
With labels: {service, method, status, endpoint}

If we have:
- 5 services
- 4 methods (GET, POST, PUT, DELETE)
- 3 statuses (200, 404, 500)
- 10 endpoints

Total combinations: 5 × 4 × 3 × 10 = 600 time series!

That's 600 different "rows" of data!
```

---

## What is PromQL?

### Definition

**PromQL** (Prometheus Query Language) is a **powerful and flexible query language** used to query and manipulate time-series data from Prometheus.

### Simple Definition

PromQL is like **SQL for metrics**. It helps you ask questions about your metrics.

### SQL vs PromQL Analogy

```
SQL (for databases):
SELECT * FROM users WHERE age > 30

PromQL (for metrics):
http_requests_total{status="200"}

SQL (aggregation):
SELECT COUNT(*) FROM orders GROUP BY customer_id

PromQL (aggregation):
sum(rate(http_requests_total[5m])) by (service)
```

### Key Capabilities of PromQL

```
✓ Select Metrics: Get specific metrics with filters
✓ Time Ranges: Query historical data (last 5 min, last 1 hour)
✓ Math Operations: Add, subtract, multiply, divide metrics
✓ Aggregations: Combine multiple metrics (sum, avg, max, min)
✓ Functions: rate(), increase(), histogram_quantile(), etc.
✓ Comparisons: Find metrics > 100, < 50, == 200, etc.
✓ Regex Matching: Use patterns for label matching
✓ Range Queries: Get data over time period, not just instant
```

### Why PromQL is Powerful

```
Simple Query:
"Give me all CPU metrics"
→ metric_name

Filtered Query:
"Give me CPU metrics only for production"
→ metric_name{environment="production"}

Rate Query:
"Show me CPU usage rate over 5 minutes"
→ rate(metric_name[5m])

Aggregated Query:
"Show me average CPU per service"
→ avg(metric_name) by (service)

Complex Query:
"Show 95th percentile latency that's > 1 second"
→ histogram_quantile(0.95, metric_name) > 1
```

---

## PromQL Basics

### Basic Syntax

```
metric_name{label1="value1", label2="value2"}[time_range]
```

### Components Explained

1. **Metric Name** - What to query
   ```
   http_requests_total
   container_cpu_usage_seconds_total
   node_memory_MemAvailable_bytes
   ```

2. **Labels** - Filter by labels
   ```
   {method="GET"}
   {status="200"}
   {namespace="production", status!="200"}
   ```

3. **Time Range** - Historical data
   ```
   [5m]   - Last 5 minutes
   [1h]   - Last 1 hour
   [7d]   - Last 7 days
   [30m]  - Last 30 minutes
   ```

---

## Selecting Time Series

### Query 1: All Metrics with Same Name

```promql
container_cpu_usage_seconds_total
```

**What it does:**
Returns all `container_cpu_usage_seconds_total` metrics (all labels)

**Result:**
```
container_cpu_usage_seconds_total{namespace="default", pod="api-1"} 45.23
container_cpu_usage_seconds_total{namespace="default", pod="api-2"} 38.12
container_cpu_usage_seconds_total{namespace="kube-system", pod="coredns-1"} 12.45
container_cpu_usage_seconds_total{namespace="kube-system", pod="coredns-2"} 14.67
```

### Query 2: Filter by Single Label

```promql
container_cpu_usage_seconds_total{namespace="kube-system"}
```

**What it does:**
Returns only from `kube-system` namespace

**Result:**
```
container_cpu_usage_seconds_total{namespace="kube-system", endpoint="https-metrics"} 45.23
container_cpu_usage_seconds_total{namespace="kube-system", pod="coredns"} 12.45
```

### Query 3: Filter by Multiple Labels

```promql
container_cpu_usage_seconds_total{namespace="kube-system", pod=~"kube-proxy.*"}
```

**What it does:**
- Namespace equals "kube-system"
- Pod name matches regex pattern "kube-proxy.*" (starts with kube-proxy)

**Operators:**

```
=   Exact match
!=  Not equal
=~  Regex match
!~  Regex not match
```

**Examples:**

```promql
# Exact match
{method="GET"}

# Not equal
{status!="200"}

# Regex match (ends with .com)
{hostname=~".*\.com"}

# Regex not match
{environment!~"(staging|dev)"}
```

### Query 4: Multiple Conditions with AND

```promql
http_requests_total{method="GET", status="200", service="api"}
```

**What it does:**
All three conditions must be true

---

## Range Vectors

### What are Range Vectors?

Range vectors return **data over a time period**, not just a single point.

### Syntax

```promql
metric_name{labels}[time_range]
```

The `[time_range]` at the end makes it a range vector.

### Examples

```promql
# Last 5 minutes of data
container_cpu_usage_seconds_total[5m]

# Last 1 hour
node_memory_MemAvailable_bytes[1h]

# Last 30 minutes
http_requests_total{status="200"}[30m]
```

### What Does It Return?

```
For a single metric over 5 minutes:

container_cpu_usage_seconds_total[5m]

Time      Value
10:25     42.00
10:26     42.50
10:27     43.00
10:28     43.50
10:29     44.00
10:30     45.23 (current)

(6 data points, one per minute)
```

### Why Range Vectors Are Useful

Use for:
- `rate()` - Calculate per-second rate
- `increase()` - Total increase over period
- `avg_over_time()` - Average over time
- Any function requiring history

---

## Aggregation Operators

### What is Aggregation?

Aggregation **combines multiple time series** into one, based on labels.

**SQL Analogy:**
```sql
SELECT COUNT(*) FROM orders GROUP BY customer_id
```

**PromQL Equivalent:**
```promql
sum(http_requests_total) by (service)
```

### Aggregation Operators

```
sum()      - Add all values together
avg()      - Calculate average
min()      - Find minimum value
max()      - Find maximum value
count()    - Count number of series
stddev()   - Standard deviation
topk()     - Top K values
bottomk()  - Bottom K values
quantile() - Calculate percentile
group()    - Just group by label
```

### Example 1: Sum CPU Usage Across All Nodes

```promql
sum(rate(node_cpu_seconds_total[5m]))
```

**What it does:**
1. `node_cpu_seconds_total[5m]` - Get 5-minute history
2. `rate()` - Convert to per-second
3. `sum()` - Add them all up

**Result:**
One single value: total CPU usage across entire cluster

### Example 2: Average Memory Usage per Namespace

```promql
avg(container_memory_usage_bytes) by (namespace)
```

**What it does:**
1. Get all memory metrics
2. Group by namespace
3. Calculate average for each namespace

**Result:**
```
namespace="default": 512MB average
namespace="kube-system": 256MB average
namespace="production": 1024MB average
```

### Example 3: Top 5 Pods by Memory Usage

```promql
topk(5, container_memory_usage_bytes)
```

**What it does:**
Returns the 5 pods using the most memory

**Result:**
```
pod-1: 1GB
pod-2: 900MB
pod-3: 800MB
pod-4: 700MB
pod-5: 600MB
```

### Grouping with `by` and `without`

#### Group By Specific Labels

```promql
sum(http_requests_total) by (service)
```

Result grouped by service only:
```
service="api": 5000
service="checkout": 3000
service="payment": 2000
```

#### Exclude Specific Labels

```promql
sum(http_requests_total) without (instance)
```

**What it does:**
Group by everything EXCEPT instance label

### Aggregation with Multiple Grouping

```promql
sum(http_requests_total) by (service, status)
```

Result grouped by service AND status:
```
service="api", status="200": 4950
service="api", status="500": 50
service="checkout", status="200": 2900
service="checkout", status="500": 100
```

---

## PromQL Functions

### Rate Function

**Purpose:** Calculate per-second rate of change

**Syntax:**
```promql
rate(metric_name[time_range])
```

**Example:**

```promql
rate(http_requests_total[5m])
```

**What it does:**
- Take 5-minute history
- Calculate how many requests per second on average
- Smooths out spikes

**Result:**
```
If 600 requests in 5 minutes (300 seconds):
Rate = 600 / 300 = 2 requests per second
```

### Increase Function

**Purpose:** Total increase over time period

**Syntax:**
```promql
increase(metric_name[time_range])
```

**Example:**

```promql
increase(container_cpu_usage_seconds_total[1h])
```

**What it does:**
- Show total increase in counter over 1 hour
- Useful for seeing "how much changed"

**Result:**
```
If counter was 100 at start and 250 at end:
Increase = 250 - 100 = 150
```

### Histogram Quantile Function

**Purpose:** Calculate percentiles from histogram metrics

**Syntax:**
```promql
histogram_quantile(quantile, histogram_metric)
```

**Example:**

```promql
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

**What it does:**
- Calculate 95th percentile latency
- Means 95% of requests are faster than this value
- Only 5% of requests take longer

**Useful Percentiles:**

```
0.50  - 50th percentile (median, 50% below, 50% above)
0.90  - 90th percentile (fast, only 10% slower)
0.95  - 95th percentile (good SLA target)
0.99  - 99th percentile (very fast)
0.999 - 99.9th percentile (excellent)
```

**Result Example:**
```
If 95th percentile latency = 500ms:
Meaning: 95% of requests complete in ≤ 500ms
         5% of requests take > 500ms
```

### Math Operations

```promql
# Multiply by 100 (convert to percentage)
(100 - avg(cpu_idle_seconds)) * 100

# Divide memory used by total memory
(1 - (available_memory / total_memory)) * 100

# Subtract baseline
current_value - baseline_value

# Add two metrics
metric1 + metric2
```

### Other Common Functions

```
abs()          - Absolute value
ceil()         - Round up
floor()        - Round down
round()        - Round to nearest
sqrt()         - Square root
log()          - Natural logarithm
exp()          - e^x

min_over_time()   - Minimum value over time
max_over_time()   - Maximum value over time
avg_over_time()   - Average over time
sum_over_time()   - Sum over time
count_over_time() - Count over time

changes()      - Number of times value changed
delta()        - Difference between values
deriv()        - Derivative (rate of change)

label_replace() - Add/modify labels
label_join()    - Combine labels
```

---

## Real-World Examples

### Example 1: CPU Usage Percentage

```promql
(1 - avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m]))) * 100
```

**What it does:**
1. Get CPU idle time
2. Calculate immediate rate over 5 minutes
3. Subtract from 1 (convert idle to usage)
4. Multiply by 100 (convert to percentage)
5. Average per instance

**Result:**
```
instance="server-1": 45%
instance="server-2": 32%
instance="server-3": 78%
```

### Example 2: Memory Usage Percentage per Namespace

```promql
sum(container_memory_usage_bytes) by (namespace) / sum(container_spec_memory_limit_bytes) by (namespace) * 100
```

**What it does:**
1. Sum memory used per namespace
2. Sum memory limit per namespace
3. Divide used by limit
4. Multiply by 100 for percentage

**Result:**
```
namespace="default": 65%
namespace="kube-system": 48%
namespace="production": 87%
```

### Example 3: Error Rate

```promql
(sum(rate(http_requests_total{status=~"5.."}[5m])) by (service) / sum(rate(http_requests_total[5m])) by (service)) * 100
```

**What it does:**
1. Count 5xx errors (status 500-599)
2. Count all requests
3. Calculate percentage

**Result:**
```
service="api": 0.5% errors
service="checkout": 2.1% errors
service="payment": 0.1% errors
```

### Example 4: Pod Restart Count Over Last Hour

```promql
increase(kube_pod_container_status_restarts_total[1h])
```

**What it does:**
Show how many times containers restarted in last hour

**Result:**
```
pod-1: 0 restarts
pod-2: 5 restarts (concerning!)
pod-3: 1 restart
```

### Example 5: 95th Percentile Request Latency

```promql
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
```

**What it does:**
Show 95th percentile latency (good SLA target)

**Result:**
```
95th percentile: 850ms
Meaning: 95% of requests complete in ≤ 850ms
```

### Example 6: Requests per Second by Endpoint

```promql
sum(rate(http_requests_total[5m])) by (endpoint)
```

**What it does:**
Show request rate for each API endpoint

**Result:**
```
endpoint="/api/users": 100 requests/sec
endpoint="/api/orders": 85 requests/sec
endpoint="/health": 10 requests/sec
```

### Example 7: Top 10 Slowest Endpoints

```promql
topk(10, histogram_quantile(0.99, http_request_duration_seconds_bucket))
```

**What it does:**
Show 10 endpoints with highest 99th percentile latency

### Example 8: Disk Usage by Mount Point

```promql
(node_filesystem_size_bytes - node_filesystem_available_bytes) / node_filesystem_size_bytes * 100
```

**What it does:**
Calculate disk usage percentage

**Result:**
```
/: 65%
/data: 89%
/logs: 45%
```

---

## Best Practices

### 1. Use Meaningful Metric Names

```
✅ GOOD:
  - http_requests_total
  - container_cpu_usage_seconds_total
  - database_query_duration_seconds

❌ BAD:
  - requests
  - cpu
  - queries
```

### 2. Keep Labels Low-Cardinality

```
✅ LOW CARDINALITY (Limited values):
  - method: GET, POST, PUT, DELETE (4 values)
  - status: 200, 400, 500 (3 values)
  - environment: prod, staging (2 values)

❌ HIGH CARDINALITY (Many values):
  - user_id: 1,000,000 different users
  - request_id: Unique for every request
  - timestamp: Always changing
```

### 3. Plan Your Label Schema First

```yaml
Services Metrics:
  Labels: [service, instance, method, status, endpoint]

Database Metrics:
  Labels: [service, instance, database, query_type]

Infrastructure Metrics:
  Labels: [instance, device, mode]
```

### 4. Use Consistent Label Names

```
Always use same names across all metrics:
✅ service (not svc, service_name)
✅ instance (not server, host)
✅ method (not http_method, verb)
✅ status (not http_status, code)
```

### 5. Avoid Cardinality Explosions

```
Example: Using user_id as label
- Metric: requests{user_id="123", method="GET", status="200"}
- Metric: requests{user_id="124", method="GET", status="200"}
- Metric: requests{user_id="125", method="GET", status="200"}
- ... (1 million user IDs!)

Problem: 1 million time series = huge storage!

Solution: Remove user_id label, use application logs for user-specific data
```

### 6. Optimize Queries

```
❌ SLOW - Returns all metrics:
node_memory_MemAvailable_bytes

✅ FAST - Filter early:
node_memory_MemAvailable_bytes{environment="production"}

❌ SLOW - Complex calculation:
sum(rate(metric[5m])) / count(metric[5m]) * 100

✅ FAST - Pre-calculate in app:
calculated_percentage_metric
```

### 7. Use Appropriate Time Ranges

```
Real-time dashboard: [1m] or [5m]
Performance analysis: [5m] or [15m]
Historical trends: [1h] or [1d]
Capacity planning: [7d] or [30d]

Too short: Noisy, spiky data
Too long: Might miss recent changes
```

### 8. Test Queries Before Production

```
Test in Prometheus UI first:
1. Execute query
2. Check results make sense
3. Check if it's too slow
4. Then add to dashboards/alerts
```

---

## PromQL Query Cheat Sheet

### Selection Queries

```promql
# All metrics with this name
metric_name

# With label filter
metric_name{label="value"}

# Multiple labels (AND)
metric_name{label1="value1", label2="value2"}

# Regex match
metric_name{label=~".*pattern.*"}

# Not regex
metric_name{label!~"^test.*"}
```

### Range Queries

```promql
# Last 5 minutes
metric_name[5m]

# Last hour
metric_name[1h]

# Last 7 days
metric_name[7d]
```

### Rate Functions

```promql
# Per-second rate
rate(metric_name[5m])

# Immediate rate
irate(metric_name[5m])

# Total increase
increase(metric_name[1h])
```

### Aggregations

```promql
# Sum all
sum(metric_name)

# Sum by label
sum(metric_name) by (label)

# Average
avg(metric_name) by (label)

# Max
max(metric_name) by (label)

# Top K
topk(5, metric_name)

# Bottom K
bottomk(5, metric_name)
```

### Percentiles

```promql
# 95th percentile
histogram_quantile(0.95, metric_name)

# 99th percentile
histogram_quantile(0.99, metric_name)

# 99.9th percentile
histogram_quantile(0.999, metric_name)
```

### Math Operations

```promql
# Multiply
metric_name * 100

# Divide
metric1 / metric2

# Add
metric1 + metric2

# Subtract
metric1 - metric2

# Comparison
metric_name > 50
metric_name < 100
```

---

## Conclusion

### Key Takeaways

1. **Metrics** = Measurements (CPU%, memory, requests)
2. **Labels** = Dimensions (service, status, method)
3. **PromQL** = Query language to analyze metrics

### Metric + Label + PromQL = Powerful Monitoring

```
Metric: http_requests_total
Label: {status="200"}
PromQL: rate(http_requests_total{status="200"}[5m])

Result: How many successful requests per second
```

### Practice Path

1. Start with simple queries: `metric_name`
2. Add filters: `metric_name{label="value"}`
3. Add time ranges: `metric_name[5m]`
4. Add functions: `rate(metric_name[5m])`
5. Add aggregations: `sum(...) by (label)`
6. Combine everything for powerful insights!

---

## Quick Reference

| Task | Query |
|------|-------|
| Get metric | `metric_name` |
| Filter by label | `{label="value"}` |
| Last 5 min data | `[5m]` |
| Per-second rate | `rate(...[5m])` |
| Sum all | `sum(...)` |
| Group by label | `by (label)` |
| Top 10 | `topk(10, ...)` |
| 95th percentile | `histogram_quantile(0.95, ...)` |
