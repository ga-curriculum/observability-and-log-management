# Observability and Log Management

**Session Time:** 120 minutes

---

## Module Overview

Develop observability skills through metrics, logs, and traces. Transitioning from a prototype to production can be challenging, and moving a model into the deployment and monitoring phase requires robust oversight. Learn how to use monitoring tools to detect anomalies, visualise performance, and maintain transparency across AI pipelines.

---

## Session Breakdown

| Segment | Topic | Duration (minutes) |
|---|---|---|
| 1 | Introduction & The Three Pillars of Observability | 25 |
| 2 | Collecting and Interpreting System Data | 20 |
| 3 | Evaluating Dashboards & Detecting Anomalies | 15 |
| 4 | Lab: Monitoring Dashboard Setup | 60 |

---

## Table of Contents
1. The Three Pillars of Observability
2. Collecting and Interpreting System Data
3. Evaluating Dashboards & Detecting Anomalies
4. Lab Reference: Monitoring Dashboard Setup
5. Wrap-Up and Reflection

---

## 1. The Three Pillars of Observability

**Learning Objective:** Identify the components of observability: metrics, logs, and traces.

In complex AI architectures, simply knowing that a system has failed is not enough; engineers must understand *why* it failed. Observability is the measure of how well internal states of a system can be inferred from knowledge of its external outputs. It relies on three foundational pillars:

1. **Metrics:**
   <br/>Numeric representations of data measured over intervals of time. They are lightweight and excellent for triggering alerts.
   
   In an AI context, this includes hardware metrics (CPU/RAM) and model-specific metrics like prediction latency or drift.<br/><br/>
2. **Logs:** 
   <br/>Immutable, timestamped records of discrete events that happened within the system. 
   
   They provide the high-fidelity context needed for debugging (e.g., error stack traces).<br/><br/>
3. **Traces:**
   <br/>Representations of the end-to-end journey of a single request as it moves through a distributed system. <br/><br/>



### **Practical Example: <br/>Fraud Detection Pipeline**
Imagine a user swipes a credit card, triggering your deployed fraud detection model.
* **Metric:** <br/>The system records a latency of `850ms` (which is dangerously high).<br/><br/>
* **Trace:** <br/>The trace shows that out of the `850ms`, `50ms` was spent in the API gateway, `700ms` was spent querying the Feature Store for the user's transaction history, and `100ms` was spent on model inference.<br/><br/>
* **Log:** <br/>You investigate the logs for the Feature Store service and see: `[ERROR] 10:45:01 - Connection timeout to database cluster A, retrying on cluster B`. <br/><br/>

> *Result:* 
> <br/>You instantly know the AI model is fine, but the database infrastructure is degrading.

---

## 2. Collecting and Interpreting System Data

**Learning Objective:** Use monitoring tools to collect and interpret system data.

To effectively manage the final stage of the ML workflow—Deployment & Monitoring—teams must utilize specialized tools to gather system data continuously. 

**Prometheus** and **Grafana** are industry-standard, open-source tools used for this exact purpose. 

**Prometheus** operates on a "pull" model, actively scraping metrics from your configured AI services (like a API inference endpoint) at regular intervals. 

**Grafana** then connects to Prometheus to interpret and visualize this time-series data.

```mermaid
graph LR
    %% Define styles
    classDef ai fill:#f9f0ff,stroke:#722ed1,stroke-width:2px,color:#000;
    classDef prom fill:#f6ffed,stroke:#52c41a,stroke-width:2px,color:#000;
    classDef graf fill:#fff7e6,stroke:#fa8c16,stroke-width:2px,color:#000;

    subgraph AI_Application_Tier
        API[ML Inference API<br/>/metrics endpoint]:::ai
        DataPrep[Data Preprocessing Worker<br/>/metrics endpoint]:::ai
    end

    Prometheus[(Prometheus<br/>Time-Series DB)]:::prom
    Grafana[Grafana<br/>Dashboards]:::graf

    %% Connections
    Prometheus -->|Scrapes metrics every 10s| API
    Prometheus -->|Scrapes metrics every 10s| DataPrep
    Grafana -->|Queries PromQL| Prometheus

```

### **Practical Example: <br/>Tracking Model Confidence**
You deploy a classifier model. You configure your Python application to expose a custom metric called `ml_prediction_confidence_score`. 

Prometheus scrapes this data every 15 seconds. 

Over time, you use Grafana to interpret this data, noticing that the average confidence score drops from 0.89 to 0.65 on weekends, indicating a potential shift in user behavior that your model was not trained to handle.

---

## 3. Evaluating Dashboards & Detecting Anomalies

**Learning Objective:** Critically evaluate observability dashboards to detect anomalies and improve transparency.

A dashboard is only as valuable as the actions it drives. Just as choosing the right evaluation metric is critical for model training (e.g., using Precision/Recall instead of Accuracy for imbalanced datasets), choosing the right operational metrics is critical for dashboard transparency.

When evaluating dashboards, engineers must look for anomalies—patterns that deviate from expected behavior. Understanding when metrics might be misleading is crucial for proper evaluation. 
> For example, looking at the *average* (mean) latency can hide massive outliers; instead, engineers should evaluate the 95th or 99th percentile (p95/p99) to see the true experience of the slowest requests.

### **Practical Example: <br/>The "Silent" Failure**
You are reviewing a Grafana dashboard for a recommendation engine. The `HTTP 500 Error Rate` panel shows a perfect 0%. The system looks healthy. However, you critically evaluate the `Incoming Request Volume` panel and notice traffic has dropped by 90% in the last hour. 

The model isn't failing; the upstream application sending data to the model is broken. By evaluating multiple panels contextually, you detect an anomaly that a single metric would have missed.

---

## 4. Lab Reference: Monitoring Dashboard Setup

**Lab Objective:** Collect and visualise live metrics with Prometheus and Grafana. This hands-on lab builds foundational skills in monitoring system performance and detecting anomalies.

*Note: Please refer to the dedicated Lab Guide repository for the complete instructions, Docker Compose files for spinning up the Prometheus/Grafana stack, and the mock ML service to monitor.*

**Tools Required:**

* Prometheus (open-source)
* Grafana (open-source)

---

## Wrap-Up and Reflection

**Reflection Questions:**

1. If your machine learning model is generating highly inaccurate predictions but the API is functioning perfectly (low latency, no HTTP errors), which of the three pillars of observability (Metrics, Logs, Traces) would be most useful in identifying the root cause, and why?
2. How does continuous observability reduce the business risk associated with deploying new AI models to production?

---

## Resources

* [The Distributed Tracing Guide (OpenTelemetry)](https://opentelemetry.io/docs/concepts/observability-primer/)
* [Prometheus: Getting Started](https://prometheus.io/docs/prometheus/latest/getting_started/)
* [Grafana Fundamentals](https://grafana.com/tutorials/grafana-fundamentals/)
