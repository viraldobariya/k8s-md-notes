# 🟢 Horizontal Pod Autoscaler (HPA) — Complete In-Depth Notes (Clear + Practical)

> **HPA automatically scales the number of Pods** in a Deployment/StatefulSet based on resource usage or metrics.

If load increases → Pods increase
If load decreases → Pods decrease

---

# 1️⃣ Why HPA is Needed?

Without HPA:

* Traffic spike → App crashes ❌
* Low traffic → Wasting resources ❌

With HPA:

* Auto scale up/down ✅
* Better performance ✅
* Cost optimization ✅

---

# 2️⃣ What HPA Scales

HPA works with:

* Deployment
* StatefulSet
* ReplicaSet

It adjusts:

```id="z5m1a8"
spec.replicas
```

It does NOT scale:

* DaemonSet
* Jobs
* Nodes (that’s Cluster Autoscaler)

---

# 3️⃣ How HPA Works Internally

Flow:

```text id="qzn19k"
Metrics Server
     ↓
HPA Controller
     ↓
Compare current vs desired metrics
     ↓
Increase / Decrease replicas
```

HPA requires:

```id="2q3clz"
metrics-server installed
```

---

# 4️⃣ Basic HPA Example (CPU-Based Scaling)

## Step 1 — Deployment with Resource Requests (Very Important)

```yaml id="3j0ix3"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          resources:
            requests:
              cpu: "100m"
            limits:
              cpu: "200m"
```

⚠️ HPA needs **CPU requests** defined.

---

## Step 2 — Create HPA

```yaml id="4nuj7z"
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 1
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

---

# 5️⃣ What This Means

If:

```id="8kq7xa"
Average CPU usage > 50%
```

Then:

```id="og1m2s"
Scale up Pods
```

If CPU drops below 50% → Scale down.

---

# 6️⃣ Scaling Formula (Concept)

HPA calculates:

```id="9k3axn"
desiredReplicas = currentReplicas × ( currentMetric / targetMetric )
```

Example:

* Current replicas = 2
* CPU usage = 80%
* Target = 50%

```id="tdif5s"
2 × (80/50) = 3.2 ≈ 4 replicas
```

---

# 7️⃣ Types of Metrics (Important)

HPA supports 3 metric types:

| Type     | Example                                  |
| -------- | ---------------------------------------- |
| Resource | CPU, Memory                              |
| Pods     | Custom per-Pod metrics                   |
| External | Cloud metrics (requests/sec, queue size) |

---

# 8️⃣ CPU vs Memory

### CPU (Most Common)

```id="c7fc2k"
averageUtilization
```

### Memory

```id="v9bt3l"
averageValue
```

Example:

```yaml id="q5h9ru"
resource:
  name: memory
  target:
    type: AverageValue
    averageValue: 500Mi
```

---

# 9️⃣ Real Production Example

Imagine:

* Backend API
* High traffic spikes
* Connects to DB like PostgreSQL

Traffic increases:

```id="wrv6cz"
CPU increases
```

HPA:

```id="n9amyk"
Scales Pods from 2 → 6
```

Traffic drops:

```id="du7ywd"
Scales back to 2
```

---

# 🔟 HPA with Multiple Metrics

You can define multiple metrics:

```yaml id="l3j16q"
metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60

  - type: Resource
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 500Mi
```

HPA chooses the **highest required replica count**.

---

# 1️⃣1️⃣ Commands

Create HPA quickly:

```bash id="7r5tzl"
kubectl autoscale deployment nginx-deployment \
  --cpu-percent=50 \
  --min=1 \
  --max=5
```

Check HPA:

```bash id="t3v1nb"
kubectl get hpa
```

Describe:

```bash id="68xj3k"
kubectl describe hpa nginx-hpa
```

---

# 1️⃣2️⃣ Important Conditions

HPA needs:

✔ Metrics Server installed
✔ Resource requests defined
✔ Deployment already running

Check metrics:

```bash id="lf3ix6"
kubectl top pods
```

---

# 1️⃣3️⃣ Common Mistakes

❌ Not defining CPU requests
❌ Metrics server not installed
❌ Setting minReplicas = maxReplicas
❌ Scaling DB StatefulSet blindly
❌ Using HPA without readiness probes

---

# 1️⃣4️⃣ HPA vs VPA vs Cluster Autoscaler

| Feature                | HPA | VPA | Cluster Autoscaler |
| ---------------------- | --- | --- | ------------------ |
| Scales Pods            | ✅   | ❌   | ❌                  |
| Scales CPU/Memory size | ❌   | ✅   | ❌                  |
| Scales Nodes           | ❌   | ❌   | ✅                  |

---

# 1️⃣5️⃣ Real Architecture Flow

```text id="n94rse"
User Traffic ↑
     ↓
CPU usage ↑
     ↓
Metrics Server detects
     ↓
HPA scales Deployment
     ↓
More Pods created
     ↓
Load balanced by Service
```

---

# 1️⃣6️⃣ When NOT to Use HPA

❌ For stateful databases
❌ If app startup time is very slow
❌ If no resource limits defined
❌ If scaling must be manual

---

# 🧠 Final Understanding

HPA:

> Automatically adjusts number of Pods based on load.

It improves:

* Availability
* Performance
* Cost efficiency

---

# 🎯 One-Line Summary

> HPA = Automatic horizontal scaling based on metrics.

---

If you want next:

* 🔥 HPA + Ingress real traffic example
* 🔥 HPA interview questions
* 🔥 Custom metrics with Prometheus
* 🔥 Deep internal controller working explanation
