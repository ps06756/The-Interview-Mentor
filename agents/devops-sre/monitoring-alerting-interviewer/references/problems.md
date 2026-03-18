# Problem Bank: Monitoring & Alerting

## Problem 1: Design Monitoring for a Payment Service

### Problem Statement
You are the new SRE for a payment service that processes credit card charges, refunds, and communicates with three external processors (Stripe, PayPal, Adyen). The service has zero monitoring today. Developers find out about issues when customers tweet complaints. Design a complete monitoring strategy.

### Solution Walkthrough

**Layer 1: The Four Golden Signals**

| Signal | What to Measure | How to Measure | Alert Threshold |
|--------|----------------|----------------|-----------------|
| **Latency** | Time to process a payment | Histogram: `payment_duration_seconds` with labels for endpoint and processor | p99 > 2s for 5 min |
| **Traffic** | Payments per second | Counter: `payment_requests_total` | Anomaly: < 50% of expected traffic for time of day |
| **Errors** | Failed payment rate | Counter: `payment_errors_total` with label for error type | Error rate > 1% for 2 min |
| **Saturation** | Resource utilization | Gauges: DB connection pool usage, thread pool usage, CPU, memory | DB pool > 80% for 5 min |

**Layer 2: Per-Processor Metrics**

Each external processor should be tracked independently:
```
payment_processor_latency_seconds{processor="stripe", endpoint="/v1/charges"}
payment_processor_latency_seconds{processor="paypal", endpoint="/v2/payments"}
payment_processor_latency_seconds{processor="adyen", endpoint="/payments"}

payment_processor_errors_total{processor="stripe", error_type="timeout"}
payment_processor_errors_total{processor="paypal", error_type="5xx"}
```

Why independently? If Stripe is slow but PayPal and Adyen are fine, the aggregate latency might look acceptable, but Stripe customers are having a terrible experience.

**Layer 3: Business Metrics**

```
payment_amount_dollars{currency="USD", status="completed"}
payment_amount_dollars{currency="USD", status="refunded"}
payment_chargebacks_total
payment_processor_fallback_total   # Times we routed away from a degraded processor
```

These metrics matter to the business and can reveal issues that technical metrics miss. A drop in payment_amount_dollars with a stable request rate might mean customers are abandoning carts.

**Layer 4: Dependency Health**

```
database_connection_pool_active
database_connection_pool_max
database_query_duration_seconds{query_type="charge_insert"}
redis_cache_hit_ratio
redis_connection_errors_total
```

**Dashboard Layout (Grafana):**

```
Row 1: Golden Signals (4 panels)
  [Request Rate] [Error Rate] [p50/p95/p99 Latency] [Saturation]

Row 2: Per-Processor Health (3 panels)
  [Stripe Latency + Errors] [PayPal Latency + Errors] [Adyen Latency + Errors]

Row 3: Business Metrics (3 panels)
  [Payment Volume $] [Success Rate %] [Refund Rate %]

Row 4: Dependencies (3 panels)
  [DB Connection Pool] [DB Query Latency] [Redis Hit Rate]
```

### Follow-up Questions
- "Stripe is having an outage. How does your monitoring help you decide to route traffic to PayPal instead?" -- Per-processor error rate alert triggers. If Stripe error rate > 10%, the circuit breaker routes to PayPal. The `payment_processor_fallback_total` metric tracks this.
- "How do you know if a drop in traffic is because of your service or because customers are just buying less?" -- Compare traffic against business metrics (marketing campaigns, time of day patterns). Use anomaly detection based on historical baselines.

---

## Problem 2: Reduce Alert Fatigue

### Problem Statement
Your team gets 200 alerts per week. The on-call rotation is dreaded -- engineers lose sleep over false positives and miss real incidents buried in noise. Morale is dropping. The last quarterly review showed that only 15% of alerts resulted in human action. Fix this.

### Solution Walkthrough

**Step 1: Alert Audit (Week 1)**

Export all alerts from the last 30 days and categorize them:

| Category | Count | % | Action |
|----------|-------|---|--------|
| Auto-resolved (< 5 min) | 85 | 42% | Convert to metric, remove alert |
| Known issue, no action taken | 45 | 23% | Fix root cause or remove alert |
| Duplicate (same incident, multiple alerts) | 30 | 15% | Deduplicate with alert grouping |
| Actioned by human | 30 | 15% | Keep, refine thresholds |
| False positive (bug in alert rule) | 10 | 5% | Fix the alert rule |

**Step 2: Delete Aggressively (Week 2)**

Apply this rule to every alert:

"If nobody took action on this alert in the last 30 days, delete it."

This typically removes 50-70% of alerts immediately.

**Step 3: Tier and Route (Week 3)**

```
P1 (Page immediately):
  - Payment success rate < 95% for 2 minutes
  - Service completely down (0 requests)
  - Error budget burn rate > 14x for 5 minutes
  Requirements: Runbook linked, auto-creates incident channel

P2 (Page during business hours):
  - Error budget burn rate > 6x for 30 minutes
  - Single processor degraded
  Requirements: Runbook linked

P3 (Slack #alerts, review within 4 hours):
  - Error budget burn rate > 3x for 6 hours
  - Disk usage > 80%
  - Certificate expiry < 30 days

P4 (Auto-create Jira ticket):
  - Slow query detected (p99 > 1s)
  - Dependency version out of date
  - Non-critical resource warning
```

**Step 4: Require Runbooks (Week 4)**

Every P1 and P2 alert must link to a runbook that contains:
1. What this alert means in plain language
2. What to check first (specific dashboard link)
3. Common root causes and fixes
4. Escalation path if the on-call cannot resolve it

If an alert does not have a runbook, it cannot be P1.

**Step 5: Replace Static Thresholds with SLO-Based Alerts (Month 2)**

Before:
```
ALERT PaymentHighErrorRate
  IF payment_errors_total / payment_requests_total > 0.01
  FOR 2m
```
Problem: Fires during every deployment, during brief network blips, and on weekends when traffic is low (10 errors out of 500 requests = 2%).

After:
```
ALERT PaymentErrorBudgetFastBurn
  IF (
    (1 - (sum(rate(payment_success_total[1h])) / sum(rate(payment_requests_total[1h]))))
    / (1 - 0.999)
  ) > 14.4
  FOR 5m
```
This only fires when you are consuming your monthly error budget 14x faster than expected -- a real incident, not a blip.

**Step 6: Measure and Track (Ongoing)**

| KPI | Before | Target | After 3 Months |
|-----|--------|--------|-----------------|
| Alerts per week | 200 | < 20 | 18 |
| % alerts actioned | 15% | > 80% | 83% |
| MTTD (time to detect real incident) | 20 min | < 5 min | 3 min |
| On-call happiness (survey) | 2/10 | 7/10 | 7/10 |

### Follow-up Questions
- "What if a manager insists on keeping a noisy alert 'just in case'?" -- Show them the data: this alert has fired 45 times in 30 days and was never actioned. It is training the team to ignore pages, which makes real incidents worse.
- "How do you prevent alert sprawl from happening again?" -- Require a review for every new alert: Must have a runbook, must specify severity, must be approved by the on-call team.

---

## Problem 3: Implement SLO-Based Alerting

### Problem Statement
Your payment service has a static threshold alert: "error rate > 1% for 2 minutes." This alert fires during deployments (brief error spike), on low-traffic nights (3 errors out of 200 requests = 1.5%), and during minor blips. But last month, it missed a slow degradation where error rate crept from 0.3% to 0.8% over two weeks, eventually causing a major incident. Replace this with SLO-based alerting.

### Solution Walkthrough

**Step 1: Define the SLO**

```
SLO: 99.9% of payment requests succeed over a 30-day rolling window.
Error budget: 0.1% of requests = 43.2 minutes of downtime/month
            = 0.1% of total request count over 30 days
```

**Step 2: Calculate Burn Rate**

Burn rate = (actual error rate) / (allowed error rate)

If SLO is 99.9%, allowed error rate is 0.1%.
If actual error rate is 1%, burn rate = 1% / 0.1% = 10x.
At 10x burn rate, you exhaust your monthly budget in 3 days.

**Step 3: Multi-Window Burn Rate Alerts**

| Alert | Burn Rate | Long Window | Short Window | Time to Exhaust Budget | Severity |
|-------|-----------|-------------|--------------|----------------------|----------|
| Fast burn | > 14.4x | 1 hour | 5 min | ~2 days | P1 (page) |
| Medium burn | > 6x | 6 hours | 30 min | ~5 days | P2 (page, business hours) |
| Slow burn | > 3x | 3 days | 6 hours | ~10 days | P3 (Slack) |

**Why multi-window?**

The long window catches sustained problems. The short window confirms the problem is happening right now (not just historical). Both must be true to fire, which dramatically reduces false positives.

Example: A 30-second deployment error spike.
- Short window (5 min): burn rate spikes to 50x
- Long window (1 hour): burn rate is 0.5x (the spike is diluted)
- Result: Alert does NOT fire. Correct behavior.

Example: A slow degradation over 2 weeks.
- Short window (6 hours): burn rate is 3.5x
- Long window (3 days): burn rate is 3.2x
- Result: P3 alert fires. The static threshold at 1% would NOT have caught this.

**Step 4: Prometheus Rules**

```yaml
groups:
- name: payment-slo-alerts
  rules:

  # Error ratio over different windows
  - record: payment:error_ratio:1h
    expr: |
      1 - (
        sum(rate(payment_requests_total{status="success"}[1h]))
        /
        sum(rate(payment_requests_total[1h]))
      )

  - record: payment:error_ratio:6h
    expr: |
      1 - (
        sum(rate(payment_requests_total{status="success"}[6h]))
        /
        sum(rate(payment_requests_total[6h]))
      )

  - record: payment:error_ratio:3d
    expr: |
      1 - (
        sum(rate(payment_requests_total{status="success"}[3d]))
        /
        sum(rate(payment_requests_total[3d]))
      )

  # Burn rate alerts
  - alert: PaymentSLOFastBurn
    expr: |
      payment:error_ratio:1h / 0.001 > 14.4
      and
      payment:error_ratio:5m / 0.001 > 14.4
    for: 2m
    labels:
      severity: P1
    annotations:
      summary: "Payment error budget burning 14x faster than expected"
      runbook: "https://runbooks.internal/payment-slo-fast-burn"

  - alert: PaymentSLOMediumBurn
    expr: |
      payment:error_ratio:6h / 0.001 > 6
      and
      payment:error_ratio:30m / 0.001 > 6
    for: 5m
    labels:
      severity: P2
    annotations:
      summary: "Payment error budget burning 6x faster than expected"

  - alert: PaymentSLOSlowBurn
    expr: |
      payment:error_ratio:3d / 0.001 > 3
      and
      payment:error_ratio:6h / 0.001 > 3
    for: 30m
    labels:
      severity: P3
    annotations:
      summary: "Payment error budget trending toward exhaustion"
```

**Step 5: Error Budget Dashboard**

```
Panel 1: Error Budget Remaining (%)
  - 30-day rolling calculation
  - Color: Green > 50%, Yellow > 25%, Red < 25%

Panel 2: Burn Rate (current)
  - Shows current 1h, 6h, 3d burn rates
  - Threshold lines at 14.4x, 6x, 3x

Panel 3: Error Budget Over Time (line chart)
  - Shows budget consumption over the last 30 days
  - Projected exhaustion date

Panel 4: Recent Budget-Consuming Events
  - Table of incidents that consumed budget
  - Duration, error rate, budget consumed
```

### Follow-up Questions
- "What if traffic patterns change significantly (e.g., Black Friday)?" -- SLO-based alerting is inherently traffic-aware because it is based on the ratio of errors to total requests, not absolute counts.
- "Should you alert on latency SLOs the same way?" -- Yes. Define a latency SLI (e.g., 99% of requests < 500ms) and apply the same burn-rate approach.

---

## Problem 4: Design Dashboards for a Microservices Platform

### Problem Statement
Your platform has 15 microservices. During incidents, engineers waste 5-10 minutes figuring out which service is the root cause because all dashboards look the same and there is no clear entry point. Design a dashboard hierarchy that guides engineers from "something is wrong" to "this is the root cause" in under 2 minutes.

### Solution Walkthrough

**Dashboard Hierarchy:**

```
Level 0: Platform Overview (the "war room" dashboard)
    |
    +-- Shows all 15 services as tiles
    |   Each tile: traffic light (green/yellow/red)
    |   Based on: SLO compliance for each service
    |
    +-- Click a red service tile
    |
    v
Level 1: Service Overview (one per service)
    |
    +-- Four Golden Signals for this service
    |   [Request Rate] [Error Rate] [Latency p50/p95/p99] [Saturation]
    |
    +-- SLO compliance and error budget remaining
    |
    +-- Dependency health (downstream services)
    |
    +-- Recent deployments timeline
    |
    +-- Click on a degraded metric
    |
    v
Level 2: Detailed Service Dashboard
    |
    +-- Per-endpoint breakdown
    |   /charge: 200ms p99, 0.1% errors
    |   /refund: 150ms p99, 0.05% errors
    |   /status: 50ms p99, 0% errors
    |
    +-- Per-dependency breakdown
    |   Stripe: 180ms, 0.1% errors
    |   Database: 5ms, 0% errors
    |   Redis: 1ms, 0% errors
    |
    +-- Resource utilization
    |   CPU, Memory, Connection pools, Thread pools
    |
    v
Level 3: Logs and Traces
    +-- Link to Grafana Loki / Kibana with pre-filtered query
    +-- Link to Jaeger / Tempo with service name pre-selected
```

**Incident Walkthrough Example:**

```
2:00 AM: Page fires for payment-service
2:01 AM: Open Level 0 dashboard
         -> payment-service: RED, order-service: YELLOW, all others GREEN
2:01 AM: Click payment-service -> Level 1
         -> Error rate: 5% (normal: 0.1%)
         -> Latency p99: 8s (normal: 200ms)
         -> Recent deployments: None in 12 hours
2:02 AM: Click error rate panel -> Level 2
         -> /charge endpoint: 8% errors (all others normal)
         -> Stripe: 15% error rate, 6s latency
         -> PayPal: normal, Adyen: normal
2:02 AM: Root cause: Stripe is degraded
2:03 AM: Check Stripe status page: Confirmed outage
2:03 AM: Action: Enable circuit breaker to route Stripe traffic to PayPal
```

Time from page to root cause: **2 minutes**.

**Design Principles:**

1. **Top-down navigation**: Always start broad, drill down to specifics
2. **Consistent layout**: Every service dashboard has the same structure
3. **Link to context**: Every panel should link to the next level of detail
4. **Show recent changes**: Overlay deployment markers on all time-series graphs
5. **Traffic light simplicity**: At a glance, red/yellow/green tells you where to focus

### Follow-up Questions
- "How do you handle 15 services all turning red at once?" -- Look at shared dependencies. If all services depend on the same database and it is down, the database dashboard (infrastructure level) should be your first stop.
- "Who builds and maintains these dashboards?" -- Each team owns their Level 1 and Level 2 dashboards. The platform team owns Level 0 and provides templates.

---

## Problem 5: On-Call Incident Response Workflow

### Problem Statement
You get paged at 3 AM: "Payment error rate > 5%." You have 5 minutes to assess the situation and decide on a course of action. Walk through your exact process, including what you look at, what commands you run, and how you communicate with stakeholders.

### Solution Walkthrough

**T+0: Alert Fires**
```
PagerDuty: CRITICAL - Payment error rate 5.2% (SLO: 0.1%)
Burn rate: 52x normal
Runbook: https://runbooks.internal/payment-high-error-rate
```

**T+0-30s: Acknowledge and Open Context**
1. Acknowledge alert in PagerDuty
2. Open the runbook (bookmark it in advance)
3. Open payment service Level 1 dashboard (bookmark)
4. Post in #incidents Slack channel: "Investigating payment error rate spike. Will update in 5 min."

**T+30s-2min: Assess the Situation**

Check the dashboard:
```
Error rate:    5.2% (normal: 0.1%)     --> CRITICAL
Latency p99:   8.5s (normal: 200ms)    --> Correlated
Request rate:  5,000/s (normal: 5,000)  --> Traffic is normal
Last deploy:   8 hours ago              --> Not a deployment issue
```

Drill into per-dependency metrics:
```
Stripe:   12% error rate, 7s latency   --> DEGRADED
PayPal:   0.1% error rate, 150ms       --> Normal
Adyen:    0.2% error rate, 180ms       --> Normal
Database: 0% errors, 5ms latency       --> Normal
```

**T+2min: Identify Root Cause**

Stripe is degraded. Check Stripe status page (statuspage.io/stripe):
"We are investigating increased API errors. Updates to follow."

**T+2-3min: Decide on Action**

Options:
1. Wait for Stripe to recover (risky -- unknown timeline)
2. Enable circuit breaker to route Stripe traffic to PayPal/Adyen (immediate)

Decision: Option 2. Customers should not suffer while Stripe investigates.

**T+3-4min: Execute**
```
# Enable circuit breaker via feature flag
curl -X POST https://feature-flags.internal/api/flags/stripe-circuit-breaker \
  -d '{"enabled": true, "reason": "Stripe outage - routing to backup processors"}'
```

**T+4-5min: Verify and Communicate**

Check dashboard:
```
Error rate:    0.3% (recovering)
Latency p99:   250ms (recovering)
Fallback rate: 40% routed to PayPal, 10% to Adyen
```

Post update:
```
#incidents: Payment errors recovering. Root cause: Stripe API degradation.
Mitigation: Circuit breaker enabled, routing to backup processors.
Error rate dropping from 5.2% to 0.3%. Monitoring.
Will disable circuit breaker when Stripe confirms resolution.
```

**T+5min: Document**

Create incident ticket with timeline, root cause, mitigation, and follow-up actions.

### Follow-up Questions
- "What if all three processors were down?" -- Escalate immediately. Queue payments for retry. Show users "Payment processing delayed" instead of an error.
- "How do you prevent this from happening again?" -- Add automated circuit breaker (not manual), add Stripe health check to your monitoring, negotiate SLAs with Stripe.

---

## Sample Session Flow

### Opening (2 minutes)
**Interviewer**: "Hey, welcome. I'm the SRE team lead. I have been on call for production systems for about ten years now. I have seen every kind of monitoring failure you can imagine -- from alert storms that made my pager unusable, to silent failures that went unnoticed for hours. Today I want to see how you think about the art of monitoring. Not the tools, but the judgment calls. Let's jump in."

### Phase 1: Golden Signals (10 minutes)
**Interviewer**: "You are the new SRE for a payment service. Day one, no monitoring exists. Customers find out about outages before we do. What are the first four metrics you instrument?"

*[Candidate discusses golden signals]*

**Interviewer**: "You mentioned latency. Are you tracking average latency or percentiles? And why does that distinction matter?"

### Phase 2: SLOs and Error Budgets (15 minutes)
**Interviewer**: "Good. Now the product manager asks you: 'Is the payment service reliable?' What is your answer, and how do you make 'reliable' measurable?"

*[Candidate discusses SLIs/SLOs]*

**Interviewer**: "You set a 99.9% SLO. It's the 20th of the month and you have used 90% of your error budget. The team wants to deploy a risky refactor. What do you tell them?"

### Phase 3: Alerting Design (10 minutes)
**Interviewer**: "Your team gets 200 alerts per week. Engineers are ignoring them. Last week a real outage went unnoticed for 20 minutes. How do you fix this?"

### Phase 4: Incident Debugging (10 minutes)
**Interviewer**: "It's 2 AM and you get paged: payment error rate above 5%. Walk me through exactly what you do in the first 5 minutes."

### Closing (3 minutes)
**Interviewer**: "Let me give you some feedback..."

*[Generate scorecard]*
