# Problem Bank: CI/CD Pipeline Design

## Problem 1: Design a CI/CD Pipeline for a Microservices Monorepo

### Problem Statement
Your team owns 8 microservices in a single monorepo. Each service has its own directory with a Dockerfile, test suite, and deployment manifests. The current pipeline builds and tests all 8 services on every push, taking 25 minutes. Developers are frustrated. Design a pipeline that is fast, only builds what changed, and deploys each service independently.

### Solution Walkthrough

**Pipeline Architecture:**

```
git push to main
    |
    v
[Detect Changed Services] (30 seconds)
    |-- git diff --name-only HEAD~1
    |-- Map changed files to service directories
    |-- Output: ["order-service", "payment-service"]
    |
    v
[Fan Out: Parallel Jobs per Changed Service]
    |
    +-- order-service pipeline --------+-- payment-service pipeline
    |   |-- Lint (15s)                 |   |-- Lint (15s)
    |   |-- Unit tests (60s)           |   |-- Unit tests (45s)
    |   |-- Build image (90s)          |   |-- Build image (90s)
    |   |-- Push to registry (30s)     |   |-- Push to registry (30s)
    |                                  |
    v                                  v
[Integration Tests] (parallel, 3 min)
    |-- Spin up ephemeral namespace
    |-- Deploy changed services + dependencies
    |-- Run cross-service contract tests
    |-- Tear down
    |
    v
[Deploy to Staging] (2 min per service, parallel)
    |-- Deploy order-service to staging
    |-- Deploy payment-service to staging
    |-- Run e2e smoke tests
    |
    v
[Deploy to Production] (canary, 10 min per service)
    |-- Canary rollout order-service
    |-- Canary rollout payment-service
    |-- Monitor and auto-promote or rollback
```

**Total pipeline time: ~8 minutes** (down from 25)

**Optimization Techniques:**

1. **Selective builds**: Only build services whose files changed
2. **Dependency caching**: Cache node_modules / pip / go mod downloads between runs
3. **Docker layer caching**: Use `--cache-from` with the previous image
4. **Test parallelization**: Split large test suites across multiple runners
5. **Shared templates**: Use reusable workflow templates (GitHub Actions) or shared libraries (Jenkins)

**Changed Service Detection (GitHub Actions example):**
```yaml
jobs:
  detect-changes:
    outputs:
      services: ${{ steps.changes.outputs.services }}
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 2
    - id: changes
      run: |
        CHANGED=$(git diff --name-only HEAD~1 | grep -oP '^services/\K[^/]+' | sort -u | jq -R -s -c 'split("\n")[:-1]')
        echo "services=$CHANGED" >> $GITHUB_OUTPUT

  build:
    needs: detect-changes
    strategy:
      matrix:
        service: ${{ fromJson(needs.detect-changes.outputs.services) }}
    steps:
    - run: echo "Building ${{ matrix.service }}"
```

### Follow-up Questions
- "What if a shared library (used by all services) changes?" -- Treat shared library changes as changes to all services. Maintain a dependency graph.
- "What about infrastructure changes (Terraform, Helm charts)?" -- Separate infrastructure pipeline with its own approval gates.

---

## Problem 2: Implement Canary Deployments with Automated Rollback

### Problem Statement
Your payment service processes 10,000 transactions per minute. Last month, a bad deployment introduced a subtle bug that caused 2% of payments to fail silently. The team did not notice for 15 minutes, costing the company $500K. Design a canary deployment process that catches bad releases before they impact more than 1% of users.

### Solution Walkthrough

**Canary Deployment Flow:**

```
Step 1: Deploy canary (1 Pod with v2)
    Stable: 9 Pods (v1) -- 95% traffic
    Canary: 1 Pod (v2)  -- 5% traffic

Step 2: Analysis (every 2 min for 10 min)
    Compare canary vs stable:
    - Error rate:           canary < stable + 0.5%
    - p99 latency:          canary < stable * 1.2
    - Payment success rate: canary >= stable - 0.1%

Step 3a: If healthy -> Promote
    5% -> 25% -> 50% -> 100%
    (2 min analysis at each step)

Step 3b: If unhealthy -> Rollback
    Route 100% to stable (v1)
    Delete canary Pod
    Alert team with analysis report
```

**Argo Rollouts Configuration:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: payment-service
spec:
  replicas: 10
  strategy:
    canary:
      canaryService: payment-canary-svc
      stableService: payment-stable-svc
      trafficRouting:
        istio:
          virtualService:
            name: payment-vs
      steps:
      - setWeight: 5
      - analysis:
          templates:
          - templateName: payment-canary-analysis
          args:
          - name: service-name
            value: payment-service
      - setWeight: 25
      - analysis:
          templates:
          - templateName: payment-canary-analysis
      - setWeight: 50
      - analysis:
          templates:
          - templateName: payment-canary-analysis
      - setWeight: 100
```

**Analysis Template:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: payment-canary-analysis
spec:
  metrics:
  - name: error-rate
    interval: 2m
    count: 5
    successCondition: result[0] < 0.005
    provider:
      prometheus:
        query: |
          sum(rate(http_requests_total{service="{{args.service-name}}",
            version="canary", status=~"5.."}[2m]))
          /
          sum(rate(http_requests_total{service="{{args.service-name}}",
            version="canary"}[2m]))
  - name: payment-success-rate
    interval: 2m
    count: 5
    successCondition: result[0] >= 0.999
    provider:
      prometheus:
        query: |
          sum(rate(payment_completed_total{service="{{args.service-name}}",
            version="canary"}[2m]))
          /
          sum(rate(payment_attempted_total{service="{{args.service-name}}",
            version="canary"}[2m]))
```

**Key Design Decisions:**
1. 5% initial canary weight limits blast radius to 500 transactions/minute
2. Statistical comparison (canary vs stable) detects relative degradation, not just absolute thresholds
3. Automated rollback removes human reaction time from the equation
4. Business metrics (payment success rate) catch bugs that technical metrics (error rate) might miss

### Follow-up Questions
- "What if the bug only manifests under high load?" -- Run load tests against the canary before increasing traffic percentage.
- "What about data corruption that isn't visible in metrics?" -- Add data integrity checks (e.g., compare order totals) as an analysis metric.

---

## Problem 3: Handle Database Migrations in a CI/CD Pipeline

### Problem Statement
Your user service is adding a `phone_number` column to the `users` table. The column will be required (NOT NULL) for new users. During a rolling deployment, both old (v1) and new (v2) versions of the service will be running simultaneously. Design a migration strategy that has zero downtime and supports rollback.

### Solution Walkthrough

**The Wrong Way (causes downtime):**
1. Run migration: `ALTER TABLE users ADD COLUMN phone_number VARCHAR(20) NOT NULL`
2. Deploy v2 that writes `phone_number`
3. Result: v1 Pods (still running during rollout) try to INSERT without `phone_number` and fail with constraint violation

**The Right Way: Expand and Contract Pattern**

**Phase 1: Expand (backward-compatible migration)**
```sql
-- Migration: 001_add_phone_number.sql
ALTER TABLE users ADD COLUMN phone_number VARCHAR(20) NULL DEFAULT NULL;
```
- Column is NULLABLE, so v1 code works fine (ignores the column)
- Deploy this migration as a pre-deploy step in the pipeline
- v1 continues running without any issues

**Phase 2: Deploy New Code**
- Deploy v2 that reads and writes `phone_number`
- v2 handles NULL `phone_number` for existing users gracefully
- v1 Pods are gradually replaced by v2 during rolling update
- Both versions coexist safely

**Phase 3: Backfill Data**
```sql
-- Run as a background job, not in the migration
UPDATE users SET phone_number = 'UNKNOWN' WHERE phone_number IS NULL;
-- Do this in batches to avoid locking:
UPDATE users SET phone_number = 'UNKNOWN'
WHERE phone_number IS NULL AND id BETWEEN 1 AND 10000;
```

**Phase 4: Contract (separate deployment, days/weeks later)**
```sql
-- Migration: 002_make_phone_number_required.sql
ALTER TABLE users ALTER COLUMN phone_number SET NOT NULL;
ALTER TABLE users ALTER COLUMN phone_number SET DEFAULT 'UNKNOWN';
```
- Only run this after ALL old code is gone and all data is backfilled
- This is a separate PR, separate deployment

**Pipeline Integration:**
```
[Pre-Deploy Stage]
    |-- Run pending database migrations
    |-- Migrations must pass backward-compatibility check:
    |   - No DROP COLUMN
    |   - No ADD NOT NULL without default
    |   - No RENAME COLUMN
    |
    v
[Deploy Stage]
    |-- Rolling update of application Pods
    |
    v
[Post-Deploy Stage]
    |-- Run data backfill jobs (if needed)
    |-- Verify migration state
```

**Rollback Considerations:**
- If v2 has a bug and you rollback to v1, the `phone_number` column still exists but is ignored by v1 -- safe.
- If you need to undo the expand migration, you can drop the nullable column -- no data loss for existing functionality.

### Follow-up Questions
- "What about column renames?" -- Never rename. Add the new column, dual-write, migrate data, drop the old column later.
- "What about large tables where ALTER TABLE locks?" -- Use online schema change tools like `gh-ost` (GitHub) or `pt-online-schema-change` (Percona).

---

## Problem 4: Speed Up a Slow CI Pipeline

### Problem Statement
Your monorepo CI pipeline takes 22 minutes. The team has 50 developers who collectively open 30 PRs per day. Each PR runs the full pipeline at least twice (initial push + after code review changes). The slow pipeline is causing PR merge queues to back up and developers to context-switch while waiting. Break down where the time is going and optimize it.

### Solution Walkthrough

**Current Pipeline Breakdown:**
```
Step                         Time
---                          ----
Checkout code                30s
Install dependencies         3 min
Lint + static analysis       2 min
Unit tests                   6 min
Build Docker images          4 min
Integration tests            5 min
Push to registry             1.5 min
                             -----
Total                        22 min
```

**Optimizations:**

**1. Dependency Caching (3 min -> 15s)**
```yaml
- uses: actions/cache@v4
  with:
    path: node_modules
    key: deps-${{ hashFiles('package-lock.json') }}
```
Only installs from scratch when `package-lock.json` changes.

**2. Parallel Lint + Unit Tests (8 min -> 3 min)**
Run lint and unit tests in parallel jobs, not sequentially.
Split unit tests across 4 runners using test sharding:
```yaml
strategy:
  matrix:
    shard: [1, 2, 3, 4]
steps:
  - run: npx jest --shard=${{ matrix.shard }}/4
```

**3. Docker Layer Caching (4 min -> 1 min)**
Use `docker buildx` with cache backend:
```yaml
- uses: docker/build-push-action@v5
  with:
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

**4. Selective Integration Tests (5 min -> 2 min)**
Only run integration tests for services that changed.
Use a dependency graph to determine which integration test suites are relevant.

**5. Merge Queue (organizational optimization)**
Use GitHub merge queue to batch PRs and run the pipeline once for a batch.

**Optimized Pipeline:**
```
Step                         Time
---                          ----
Checkout code                30s
Install dependencies (cached) 15s
Lint + Unit tests (parallel)  3 min
Build Docker images (cached)  1 min
Integration tests (selective) 2 min
Push to registry              30s
                              -----
Total                         ~7 min
```

**Impact Analysis:**
- Old: 30 PRs * 2 runs * 22 min = 1,320 pipeline-minutes/day = 22 pipeline-hours/day
- New: 30 PRs * 2 runs * 7 min = 420 pipeline-minutes/day = 7 pipeline-hours/day
- Developer wait time reduced by 68%

### Follow-up Questions
- "What about flaky tests?" -- Quarantine flaky tests: run them separately, do not block the pipeline. Track flaky test rate as a team metric.
- "How do you prevent the pipeline from getting slow again?" -- Add a CI performance budget: alert if any stage exceeds its time budget.

---

## Problem 5: Design a Rollback Strategy

### Problem Statement
You deployed v2 of your order service to production. Five minutes later, monitoring shows that 5% of orders are failing with a new error. The error was not caught by the canary because it only manifests when a specific discount code is applied, which was not exercised during the canary analysis. Design a rollback process that restores service within 2 minutes.

### Solution Walkthrough

**Immediate Rollback (< 2 minutes):**

**Option A: Kubernetes Rollback**
```
kubectl rollout undo deployment/order-service
```
- Reverts to previous ReplicaSet (v1 Pods)
- Rolling update terminates v2 Pods and starts v1 Pods
- Time: ~60-90 seconds depending on Pod startup time
- Requirement: `revisionHistoryLimit > 0` (default: 10)

**Option B: Argo Rollouts Abort**
```
kubectl argo rollouts abort order-service
```
- Immediately routes 100% of traffic to stable (v1)
- Does not destroy v2 Pods (can debug them later)
- Time: < 10 seconds (just a traffic routing change)

**Option C: Feature Flag Kill Switch**
```
// In the application code:
if (featureFlags.isEnabled('new-discount-logic')) {
  return newDiscountCalculation(order);
} else {
  return legacyDiscountCalculation(order);
}
```
- Disable the flag in LaunchDarkly/Unleash
- Takes effect within seconds (no deployment needed)
- Allows v2 to stay deployed while the specific buggy feature is disabled

**Post-Rollback Process:**

```
T+0:     Alert fires (5% order failure rate)
T+1:     On-call engineer acknowledges
T+2:     Rollback initiated (kubectl rollout undo)
T+3:     v1 Pods starting, v2 Pods draining
T+4:     Traffic fully on v1, error rate normalizing
T+5:     Incident channel opened
T+30:    Root cause identified (discount code edge case)
T+60:    Fix developed and tested locally
T+90:    Fix deployed to staging
T+120:   Fix verified in staging
T+150:   Fix deployed to production via canary
```

**Prevention for Next Time:**
1. Add the specific discount code scenario to the canary analysis test suite
2. Implement shadow testing: replay production traffic against v2 before routing real traffic
3. Use feature flags for all business logic changes to enable instant rollback without redeployment

### Follow-up Questions
- "What if the rollback also has a bug?" -- Keep multiple revision history levels. You can rollback to any previous version.
- "What about data that was written by v2?" -- Design for backward compatibility. v1 should be able to read data written by v2, or have a data remediation script ready.

---

## Sample Session Flow

### Opening (2 minutes)
**Interviewer**: "Hey, good to meet you. I'm the platform engineering lead. I've spent the last several years building CI/CD pipelines for engineering teams of all sizes. Today I want to talk about how you think about the entire lifecycle of code -- from a developer's git push all the way to production traffic. Let's get started."

### Phase 1: Fundamentals (10 minutes)
**Interviewer**: "First, let's make sure we're speaking the same language. Can you explain the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment? Most people conflate at least two of them."

*[Candidate discusses CI/CD concepts]*

**Interviewer**: "Good. Now tell me -- what does 'Continuous' actually mean in practice? Is it CI if you merge to main once a week?"

### Phase 2: Pipeline Design (15 minutes)
**Interviewer**: "OK, let's design something. You have 8 microservices in a monorepo. A developer pushes a change that touches 2 services. Walk me through the pipeline from git push to production."

*[Candidate designs pipeline]*

**Interviewer**: "This pipeline takes 20 minutes. You have 50 developers and 30 PRs a day. What is the cost of a 20-minute pipeline, and how do you cut it to under 8 minutes?"

### Phase 3: Deployment Strategies (10 minutes)
**Interviewer**: "Your payment service handles $2M/day. You need to deploy a major rewrite. What deployment strategy do you use, and walk me through the exact steps?"

### Phase 4: Edge Cases (10 minutes)
**Interviewer**: "The new version adds a required database column. How do you handle the migration without downtime, considering both old and new code versions are running during the rollout?"

### Closing (3 minutes)
**Interviewer**: "Let me give you some feedback..."

*[Generate scorecard]*
