# Problem Bank: Kubernetes Fundamentals

## Problem 1: Design a Zero-Downtime Deployment for a Payment API

### Problem Statement
You are responsible for deploying a payment processing API on Kubernetes. The API currently runs as a Deployment with 6 replicas. You need to roll out a new version that changes the database schema. Requirements: zero downtime, ability to roll back within 60 seconds, and no request failures during the transition.

### Solution Walkthrough

**Deployment Configuration:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-api
spec:
  replicas: 6
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 0
  template:
    spec:
      containers:
      - name: payment-api
        image: payment-api:v2
        readinessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          failureThreshold: 3
        livenessProbe:
          httpGet:
            path: /livez
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
      minReadySeconds: 30
```

**Key Design Decisions:**

1. `maxUnavailable: 0` -- Never remove an old Pod until a new one is ready
2. `maxSurge: 2` -- Allow 2 extra Pods during rollout for faster transitions
3. `minReadySeconds: 30` -- Wait 30 seconds after readiness before continuing
4. `revisionHistoryLimit: 5` -- Keep 5 old ReplicaSets for rollback
5. Separate readiness and liveness probes

**Database Schema Migration Strategy:**

Because the schema changes, you must use a backward-compatible migration approach:
1. Deploy schema migration as a Kubernetes Job (add new columns, do not drop old ones)
2. Deploy v2 code that reads from both old and new columns
3. Migrate data in the background
4. Deploy v3 that only uses new columns
5. Deploy schema cleanup Job to drop old columns

This is the "expand and contract" migration pattern.

**Rollback Process:**
```
kubectl rollout undo deployment/payment-api
```
This reverts to the previous ReplicaSet. Because `maxUnavailable: 0`, old Pods are still running during rollout, and rollback is near-instant.

### Follow-up Questions
- "What if the readiness probe passes but the app is silently returning wrong data?" -- Use canary deployments with traffic splitting to test with a small percentage of real traffic first.
- "What about long-running requests during Pod termination?" -- Configure `terminationGracePeriodSeconds` and handle SIGTERM in your application to drain connections.

---

## Problem 2: Debug a CrashLoopBackOff

### Problem Statement
A developer deployed a new microservice. All 3 Pods are in CrashLoopBackOff. The developer says the service runs perfectly in Docker on their laptop. The service is a Node.js API that connects to PostgreSQL and Redis.

### Solution Walkthrough

**Systematic Debugging Process:**

**Step 1: Describe the Pod**
```
kubectl describe pod api-server-abc123
```

Check the Events section:
```
Events:
  Type     Reason     Message
  ----     ------     -------
  Normal   Scheduled  Successfully assigned to node-2
  Normal   Pulled     Container image pulled successfully
  Normal   Started    Started container api-server
  Warning  BackOff    Back-off restarting failed container
```

Check Last State:
```
Last State:  Terminated
  Exit Code: 1
  Reason:    Error
```

Exit code 1 means the application itself crashed (not OOMKilled, which is 137).

**Step 2: Check Logs**
```
kubectl logs api-server-abc123 --previous
```

Output:
```
Error: connect ECONNREFUSED 10.96.0.50:5432
    at TCPConnectWrap.afterConnect
```

The application cannot reach PostgreSQL.

**Step 3: Verify the Database Service**
```
kubectl get svc postgres-svc
kubectl get endpoints postgres-svc
```

If endpoints are empty, the label selector on the Service does not match the database Pod labels.

**Common Root Causes Checklist:**

| Symptom | Exit Code | Likely Cause |
|---------|-----------|--------------|
| OOMKilled | 137 | Memory limit too low |
| Error | 1 | Application crash (missing env var, bad config, dependency unreachable) |
| Completed | 0 | Container exits normally (wrong command for long-running process) |
| ImagePullBackOff | N/A | Wrong image name, missing registry credentials |
| CrashLoopBackOff | 1 | Startup dependency not available (DB, config, secret) |

**Step 4: Check ConfigMaps and Secrets**
```
kubectl get configmap api-config -o yaml
kubectl get secret api-secrets -o yaml
```

Verify all expected environment variables are present and correct.

**Step 5: Test Connectivity from Inside the Cluster**
```
kubectl run debug --image=busybox --rm -it -- /bin/sh
# Inside the debug pod:
nslookup postgres-svc.default.svc.cluster.local
nc -zv postgres-svc 5432
```

### Follow-up Questions
- "The Pod was running fine yesterday and started crashing today. Nothing was deployed. What changed?" -- Check if a dependent service was redeployed, if a Secret/ConfigMap was updated, if a node was replaced, or if a certificate expired.

---

## Problem 3: Design Autoscaling for a Flash Sale

### Problem Statement
Your e-commerce API handles 1,000 req/s normally. During flash sales (announced 1 hour in advance), traffic spikes to 10,000 req/s within 60 seconds. Each Pod handles 200 req/s. Currently, HPA takes 5 minutes to fully scale, causing timeouts during the spike. Design a scaling strategy.

### Solution Walkthrough

**Capacity Planning:**
- Normal: 1,000 req/s / 200 per Pod = 5 Pods minimum
- Flash sale: 10,000 req/s / 200 per Pod = 50 Pods required
- Scale factor: 10x

**Multi-Layer Scaling Strategy:**

**Layer 1: Baseline Over-Provisioning**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ecommerce-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ecommerce-api
  minReplicas: 10    # 2x normal to absorb initial spike
  maxReplicas: 80    # Headroom above 50 for safety
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100          # Double Pods every 15 seconds
        periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5 min before scaling down
```

**Layer 2: Scheduled Pre-Scaling (for known events)**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: prescale-flash-sale
spec:
  schedule: "50 * * * *"    # Run at :50 of the scheduled hour
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: scaler
            image: bitnami/kubectl
            command:
            - kubectl
            - scale
            - deployment/ecommerce-api
            - --replicas=50
```

**Layer 3: Cluster Autoscaler**
- Dedicated node pool with instance type optimized for the workload
- Over-provision 2-3 "buffer" nodes that are always warm
- Use priority-based expander to prefer the dedicated pool

**Layer 4: Pod Startup Optimization**
- Use lightweight container images (distroless, Alpine)
- Pre-pull images on all nodes using a DaemonSet
- Minimize application startup time (lazy initialization, connection pooling)
- Set readiness probe `initialDelaySeconds` as low as possible

**Timeline During Flash Sale:**
```
T-10min:  CronJob scales to 50 Pods
T-5min:   All 50 Pods running and ready
T+0:      Flash sale starts, 10,000 req/s hits
T+0:      50 Pods handle load comfortably
T+30min:  Flash sale ends, traffic drops
T+35min:  HPA scaleDown stabilization window expires
T+40min:  Pods scale down to minReplicas (10)
```

### Follow-up Questions
- "What if the flash sale traffic exceeds 10,000 req/s?" -- The HPA continues scaling above 50 Pods because maxReplicas is 80, and the aggressive scaleUp policy doubles Pods every 15 seconds.
- "What about database connection limits?" -- Use a connection pooler like PgBouncer as a sidecar, and set `maxConnections` per Pod so total connections stay within the database limit.

---

## Problem 4: StatefulSet Design for a Redis Cluster

### Problem Statement
You need to deploy a 6-node Redis Cluster (3 primaries, 3 replicas) on Kubernetes. Redis nodes need stable network identities, persistent storage, and ordered startup. Design the Kubernetes resources.

### Solution Walkthrough

**Why StatefulSet, Not Deployment:**
- Redis Cluster nodes need stable hostnames (redis-0, redis-1, ...) for cluster membership
- Each node needs its own persistent storage
- Ordered startup ensures the cluster can be initialized properly

**StatefulSet Configuration:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
spec:
  serviceName: redis-cluster-headless
  replicas: 6
  podManagementPolicy: Parallel  # After initial setup
  selector:
    matchLabels:
      app: redis-cluster
  template:
    spec:
      containers:
      - name: redis
        image: redis:7
        ports:
        - containerPort: 6379
        - containerPort: 16379   # Cluster bus port
        volumeMounts:
        - name: data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 50Gi
```

**Headless Service (for stable DNS):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-cluster-headless
spec:
  clusterIP: None
  selector:
    app: redis-cluster
  ports:
  - port: 6379
    targetPort: 6379
```

Each Pod gets a DNS name: `redis-cluster-0.redis-cluster-headless.default.svc.cluster.local`

**Key Considerations:**
1. PodDisruptionBudget: Allow at most 1 Pod unavailable to maintain cluster quorum
2. Anti-affinity: Spread Pods across nodes so a single node failure does not take out a primary and its replica
3. Resource requests: Set memory request equal to limit (Guaranteed QoS) because Redis is memory-sensitive

### Follow-up Questions
- "What happens when a StatefulSet Pod is deleted?" -- It is recreated with the same name and reattached to its PVC, preserving data.
- "How do you handle a node failure?" -- The Pod is rescheduled on a healthy node and reattaches its PV (if using network-attached storage like EBS). Redis Cluster handles the failover at the application level.

---

## Problem 5: RBAC Design for a Multi-Team Cluster

### Problem Statement
Your company has 3 teams sharing one Kubernetes cluster: Frontend, Backend, and Data. Requirements:
- Each team can only deploy to their own namespace
- Only the Platform team can create namespaces, ClusterRoles, or modify node configurations
- Developers can view logs and exec into Pods in their namespace
- No team can access Secrets from another team's namespace

### Solution Walkthrough

**Namespace Setup:**
```
frontend-team (namespace)
backend-team (namespace)
data-team (namespace)
```

**Role for Developers (per namespace):**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: backend-team
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets", "daemonsets"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
- apiGroups: [""]
  resources: ["pods", "pods/log", "pods/exec", "services", "configmaps"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list", "create", "update", "patch"]
```

**RoleBinding (binds Role to team):**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: backend-developers
  namespace: backend-team
subjects:
- kind: Group
  name: backend-team
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

**ClusterRole for Platform Team:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: platform-admin
rules:
- apiGroups: [""]
  resources: ["namespaces", "nodes"]
  verbs: ["*"]
- apiGroups: ["rbac.authorization.k8s.io"]
  resources: ["clusterroles", "clusterrolebindings", "roles", "rolebindings"]
  verbs: ["*"]
```

**Security Boundaries:**
- Namespace-scoped Roles ensure a team's RoleBinding only grants access within their namespace
- Secrets are namespace-scoped, so a Role in `backend-team` cannot access Secrets in `frontend-team`
- NetworkPolicies should complement RBAC to enforce network-level isolation between namespaces

### Follow-up Questions
- "How do you prevent a developer from escalating their own privileges?" -- RBAC has a built-in escalation prevention: you cannot create a RoleBinding granting permissions you do not already have.
- "What about ServiceAccount tokens?" -- Disable auto-mounting of ServiceAccount tokens unless the Pod explicitly needs API access. Use `automountServiceAccountToken: false`.

---

## Sample Session Flow

### Opening (2 minutes)
**Interviewer**: "Hey, welcome. I'm the DevOps lead here. I've been running Kubernetes clusters in production for about five years, across AWS, GCP, and bare metal. Today we're going to talk about Kubernetes fundamentals -- not trivia, but real operational knowledge. Let's start simple and go from there."

### Phase 1: Pod Fundamentals (10 minutes)
**Interviewer**: "First question -- what is a Pod? And more importantly, why does Kubernetes schedule Pods instead of individual containers?"

*[Candidate discusses Pod abstraction]*

**Interviewer**: "Good. Now tell me -- if I have a web server container and a log-shipping sidecar container in the same Pod, what do they share? And what don't they share?"

### Phase 2: Deployments & Rollouts (15 minutes)
**Interviewer**: "Let's say you have a payment API running as a Deployment with 6 replicas. Product wants to push a new version right now. How do you ensure zero downtime during the rollout?"

*[Candidate discusses rolling updates]*

**Interviewer**: "The new version is deployed, but 2 minutes later alerts fire -- the new version has a bug. What do you do, and how fast can you fix it?"

### Phase 3: Troubleshooting (10 minutes)
**Interviewer**: "A junior developer comes to you. Their new service has all 3 Pods in CrashLoopBackOff. They say it works on their laptop. Walk me through your debugging steps."

### Phase 4: Scaling (10 minutes)
**Interviewer**: "Our API does 1,000 req/s normally but we have a flash sale coming in 2 hours that will push it to 10,000 req/s. Current autoscaling takes 5 minutes to react. What do we do?"

### Closing (3 minutes)
**Interviewer**: "Let me give you some feedback..."

*[Generate scorecard]*
