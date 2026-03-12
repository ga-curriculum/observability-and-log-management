# SME Feedback: Observability and Log Management
**Module:** Week 3, Class 5 — Observability and Log Management  
**Reviewer:** SME (AI Systems Engineering)  
**Documents Reviewed:** README.md (student-facing), Instructor Guide  

---

## Overall Assessment

This is a well-constructed module with a solid practical framing. The three-pillar structure (metrics, logs, traces) is clearly explained, the fraud detection example is concrete and memorable, and the "silent failure" concept in Section 3 is exactly right for AI systems engineering. The pacing table correctly allocates 60 minutes to the lab. The recurring structural issue — deferring to a non-existent external lab guide — appears again and is the most urgent fix.

---

## 1. Structural Issues

**Issue 1.1 — Table of Contents Appears After the Session Breakdown**  
Unlike previous modules, this README places the Table of Contents *after* the Session Breakdown table. This is likely a copy-paste ordering error. The ToC should appear before the Session Breakdown so students can navigate the document before reading the timing.

**Issue 1.2 — "Wrap-Up and Reflection" Has No Section Number**  
The body section is headed "Wrap-Up and Reflection" with no number, while every other section is numbered 1–4. The ToC lists it without a number. Should be Section 5 for consistency.

**Issue 1.3 — Lab Section Defers to External Repository (Production Blocker)**  
Section 4 states: *"Please refer to the dedicated Lab Guide repository for the complete instructions, Docker Compose files for spinning up the Prometheus/Grafana stack, and the mock ML service to monitor."* No URL or path is given. This is a production blocker and the fourth consecutive module with this issue. The lab materials produced alongside this feedback are self-contained and resolve this.

---

## 2. Alignment with Syllabus and Competency Map

| Learning Objective | In README? | In Instructor Guide? | Notes |
|---|---|---|---|
| Identify the components of observability: metrics, logs, and traces | ✅ Section 1 | ✅ Yes | Fraud detection example is excellent |
| Use monitoring tools to collect and interpret system data | ✅ Section 2 | ✅ Yes | Prometheus pull model well-explained |
| Critically evaluate dashboards to detect anomalies | ✅ Section 3 | ✅ Yes | "Silent failure" traffic drop scenario is strong |

All three LOs are fully addressed. Coverage is proportionate.

---

## 3. Instructor Guide — Mermaid Diagram Arrow Direction

The instructor guide's Mermaid diagram shows:

```
A[Scrape Target: ML Model API] -->|Exposes /metrics| B;
B((Prometheus Server)) -->|Scrapes every 15s| A;
```

The first arrow (`A → B`) labelled "Exposes /metrics" is conceptually misleading — it implies the target *pushes* to Prometheus, which is the opposite of Prometheus's pull model. The correct framing is: targets *expose* a `/metrics` endpoint; Prometheus *pulls from* it. The diagram should show only the Prometheus → target direction, or use a bidirectional arrow with clear labels distinguishing "exposes endpoint" from "scrapes endpoint."

This is particularly important because the pull vs. push distinction is flagged as a key teaching point in the instructor guide's own delivery notes.

---

## 4. Technical Content

**Issue 4.1 — No `/metrics` Endpoint Example**  
Section 2 explains that AI services expose a `/metrics` endpoint for Prometheus to scrape, and the diagram correctly shows this. However, no code example shows *how* a Python service exposes metrics. Students familiar with Prometheus will know `prometheus_client`; others will not. A four-line example using `prometheus_client` would bridge theory to practice:

```python
from prometheus_client import Counter, generate_latest
REQUEST_COUNTER = Counter("model_requests_total", "Total inference requests", ["status"])
# In your route handler:
REQUEST_COUNTER.labels(status="200").inc()
```

The lab materials include a fully instrumented `app.py` using `prometheus_client` — a forward-looking recommendation is to add a condensed version to the lecture README.

**Issue 4.2 — Counter vs. Gauge vs. Histogram Not Distinguished**  
The lecture introduces Prometheus metrics at a high level but does not distinguish between the three most common metric types: Counter (only increases), Gauge (can go up or down), and Histogram (records distributions — essential for p99 latency). The AI Prompt Bank in the instructor guide asks students to ask their AI about this distinction, which is good — but students arrive at that prompt without any mental model to evaluate the AI's answer. A one-paragraph summary in the README would help.

**Issue 4.3 — "localhost vs. service name" Issue Pre-Empted in Lab, Missing from README**  
The instructor guide's troubleshooting table correctly identifies the most common student error: typing `http://localhost:9090` instead of `http://prometheus:9090` when configuring Grafana. This error should be addressed proactively in the student-facing README, not only in the instructor troubleshooting table. The lab materials resolve this by pre-provisioning the Grafana datasource via a config file — students never encounter the issue.

---

## 5. Theory-to-Lab Alignment Recommendations

> *These are forward-looking recommendations, not criticisms. The lecture was written before the lab materials existed.*

**Recommendation 5.1 — Add `prometheus_client` Metric Types to Section 2**  
The lab's `app.py` demonstrates Counter, Histogram, and Gauge with inline comments explaining each type. Adding a condensed version of these three definitions to Section 2 would let students recognise the patterns when they inspect the running service's `/metrics` endpoint during the lab.

**Recommendation 5.2 — Add a PromQL Example to Section 2**  
The Lab 4 README introduced one PromQL query for error rate. This module is where Prometheus becomes a primary tool, but the README contains no PromQL. At minimum, showing how to compute error rate and p99 latency in PromQL would scaffold the Grafana panel tasks:

```promql
# Error rate (5xx / total)
sum(rate(novamart_requests_total{status=~"5.."}[5m])) / sum(rate(novamart_requests_total[5m]))

# p99 inference latency
histogram_quantile(0.99, sum(rate(novamart_inference_latency_seconds_bucket[5m])) by (le)) * 1000
```

**Recommendation 5.3 — Distinguish Log Levels in the Lecture**  
The lab log data uses structured JSON with `level` fields (INFO, WARN, ERROR). The lecture mentions logs as "timestamped records of discrete events" but does not describe log severity levels. Adding a brief note on INFO/WARN/ERROR conventions would scaffold the log analysis notebook's use of the `level` field.

---

## Summary of Action Items

| Priority | Issue | Type | Owner |
|---|---|---|---|
| High | Provide lab Docker Compose files and mock ML service — remove external repo reference | Fix | Content Author |
| High | Move Table of Contents above Session Breakdown | Fix | Content Author |
| High | Fix Mermaid arrow direction in instructor guide (pull model, not push) | Fix | Content Author |
| Medium | Add Section number to Wrap-Up (Section 5) | Fix | Content Author |
| Medium | Add `localhost` vs. service name note to student README | Fix | Content Author |
| Medium | Add Counter/Gauge/Histogram distinction to Section 2 | Recommendation | Content Author |
| Medium | Add PromQL examples for error rate and p99 latency to Section 2 | Recommendation | Content Author |
| Low | Add `prometheus_client` snippet to Section 2 | Recommendation | Content Author |
| Low | Add log severity level conventions (INFO/WARN/ERROR) to Section 1 | Recommendation | Content Author |
