## Q1. What is Prometheus, and why is it used?

Prometheus is an open-source monitoring and alerting tool used to collect and store metrics from
systems, applications, and Kubernetes clusters.
It helps teams monitor performance, detect issues early, and ensure system reliability.


---

## Q2. Prometheus vs CloudWatch
Prometheus is an open-source monitoring tool mainly used for Kubernetes and container-based
environments, offering powerful metric collection and querying.
CloudWatch is a fully managed AWS monitoring service used to monitor AWS resources, logs,

When to Use What? (Tell it like it is)
Use Prometheus when:

- You use Kubernetes

- You want deep metrics & control

- You prefer open-source

- You are okay managing infra

Use CloudWatch when:

- You are fully on AWS

- You want zero maintenance

- You need quick setup

- You don’t want to manage monitoring infra

---

## Q3. What is the difference between Prometheus and Grafana?

Prometheus is a monitoring system that collects, stores, and evaluates metrics and generates alerts when something goes wrong. Grafana is a visualization platform that connects to Prometheus and other data sources to display metrics in dashboards and graphs. Prometheus handles monitoring logic, while Grafana focuses only on presenting data in an easy-to-understand format.

--- 

## Q4. What is Prometheus Architecture?

1 Prometheus Server
The Prometheus server is the brain of the metric-based monitoring system. The main job of the
server is to collect the metrics from various targets using pull model.
Target is nothing but a server, pod, endpoints etc which we will look in to detail in the next topic.
The general term for collecting metrics from the targets using Prometheus is called scraping.

2 TSDB
TSDB stands for Time Series Database.
It is used to store time-based metric data such as CPU, memory, and application performance
metrics.
In monitoring tools like Prometheus, TSDB stores metrics efficiently and allows fast querying
over time

3 Service Discovery
Service Discovery is a mechanism that automatically detects and tracks services in a dynamic
environment. Prometheus uses two methods to scrape metrics from the targets.
 - Static configs: When the targets have a static IP or DNS endpoint, we can use those endpoints as
targets.
 - Sevice Discovery: In most autoscaling systems and distributed systems like Kubernetes, the target
will not have a static endpoint. In this case, that target endpoints are discovered using prometheus
service discovery and targets are added automatically to the prometheus configuration.

4 Prometheus Targets
Target is the source where Prometheus scrape the metrics. A target could be servers, services,
Kubernetes pods, application endpoints, etc.

5 Prometheus Exporters
Exporters are like agents that run on the targets. It converts metrics from specific system to format
that prometheus understands.

6 Prometheus Pushgateway
Prometheus Pushgateway is used to collect metrics from short-lived or batch jobs that cannot be
scraped directly by Prometheus.
Jobs push their metrics to the Pushgateway, and Prometheus then scrapes those metrics.

7 Prometheus Client Libraries
Prometheus Client Libraries are software libraries that can be used to instrument application code
to expose metrics in the way Prometheus understands.

8 Prometheus Alertmanager
Prometheus Alertmanager is a component that handles alerts sent by Prometheus and notifies
the right people through the right channels.

9 PromQL
PromQL is a flexible query language that can be used to query time series metrics from the
prometheus.

---

## Q5 Prometheus already shows graphs — so why do we use Grafana?

Although Prometheus provides a basic UI for viewing metrics, it is mainly used for querying and
debugging.
Grafana is used because it offers advanced dashboards, better visualization, alerting, and the
ability to combine data from multiple sources, making it suitable for production monitoring and
team usage.

---

## Q6. How to expose /metrics

To expose /metrics, the application or system must provide an HTTP endpoint that returns metrics in Prometheus format. This is usually done using exporters for system-level metrics or client libraries for application metrics. Prometheus then periodically scrapes this /metrics endpoint to collect and store the data.


---

## Q7. How do you enable authentication in Prometheus?

Prometheus does not provide built-in user authentication. In real environments, authentication is enabled by placing Prometheus behind a reverse proxy such as Nginx or through Kubernetes ingress, where authentication mechanisms like basic auth, OAuth, or SSO are enforced. TLS and client certificates can also be used to secure access, and authentication can be configured when Prometheus scrapes secured targets.


--- 

## Q8 How can Prometheus be made highly available?

Prometheus is not highly available by default.
High availability is achieved by running multiple Prometheus instances.

---

## Q9 How do you secure Prometheus endpoints?

by placing Prometheus behind a reverse proxy such as Nginx or an Ingress controller, Network-
level security such as firewalls, security groups, and Kubernetes network policies is also used to restrict access

---

## Q10. Q 10 What is the default port for Prometheus?

9090

---

## Q11. What is node_exporter in Prometheus?

node_exporter is a Prometheus exporter used to collect system-level metrics from a machine such as CPU, memory, disk, and network usage. It runs on the host, exposes these metrics through an HTTP /metrics endpoint, and Prometheus scrapes this data to monitor the health and performance of the node.

---

## Q12. Q 12 Can Prometheus monitor Windows servers?

Yes, with windows_exporter, Prometheus can collect metrics from Windows servers.

---
## Q13. What is the Prometheus API used for

The Prometheus API is used to query and retrieve metrics, alert states, and target information programmatically. It allows tools like Grafana, automation scripts, and monitoring systems to access Prometheus data using PromQL, making Prometheus usable as a backend monitoring service rather than just a UI tool.

---

## Q14. Q 14 What are Prometheus’s limitations?
Prometheus has several limitations, including lack of built-in authentication, limited long-term
storage, and no native high availability.

It follows a pull-based model, which is not ideal for short-lived jobs, and it struggles with high-
cardinality metrics.

Additionally, Prometheus focuses only on metrics and does not handle logs or traces, so it is often
used with other tools for full observability.


---

## Q15 metric vs tracing vs logs

- Metrics provide numerical data used for monitoring and alerting,
- logs provide detailed event information for debugging, and
- traces show the end-to-end flow of requests across services.

---

## Q16. Prometheus federation

Prometheus federation is a setup where one Prometheus server scrapes selected metrics from
another Prometheus server using the /federate endpoint.
It’s mainly used in large or multi-cluster environments to scale monitoring, reduce load, and
keep detailed metrics at cluster level while maintaining aggregated, high-level visibility at a
global level.


---


## Q17. Explain the term “sharding” in the context of Prometheus
Sharding in Prometheus simply means splitting the monitoring load across multiple
Prometheus servers.
Instead of one Prometheus scraping everything and getting overloaded, we divide the targets so
each Prometheus handles only a part of the workload.
This helps Prometheus scale better, improves performance, and avoids a single point of failure. 


---

## Q18.  What are Counters in Prometheus?

In Prometheus, a Counter is a metric type that represents a cumulative value which only increases over time. It is mainly used to count events such as requests, errors, or job executions. Counters reset only when the application restarts and are typically used with rate or increase functions to calculate values like requests per second.

What Counters are used for (Real Meaning)

- Counters answer questions like:

- How many requests were served?

- How many errors occurred?

- How many jobs were executed?

- How many times a pod restarted?

---

## Q19. What is the default data retention period in Prometheus?

The default data retention period is 15 days in Prometheus. Data would be automatically deleted after the data storage default retention duration has passed.

---

## Q20. What is Thanos Prometheus?

Thanos is a set of components that extends Prometheus to provide high availability, long-term storage, and a global query view. It works alongside Prometheus by uploading metrics to object storage and allowing deduplicated queries across multiple Prometheus instances, making Prometheus suitable for large-scale and multi-cluster environments.

Scenario : Multi-Cluster Kubernetes Monitoring (Most Common)
Situation

You have:

5 Kubernetes clusters

Different environments: dev, staging, prod

Each cluster runs its own Prometheus

Problem without Thanos:

Each Prometheus has its own data

Grafana dashboards must be duplicated

No single place to see overall system health

How Thanos helps

Each cluster Prometheus runs with Thanos Sidecar

Metrics are uploaded to object storage

One Thanos Query gives a global view

Grafana connects to Thanos Query

---

## Q21 What protocol does Prometheus use?

Prometheus is primarily based on a pull model, in which the prometheus server has a list of targets it should scrape metrics from. The pull protocol is HTTP based and simply put, the target returns a list of 


---

## Q22 How do you integrate with Prometheus?

