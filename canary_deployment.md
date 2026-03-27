
Here’s a **short, crisp “top-of-notes” section** you can prepend to your Canary notes:

---

## 🚦 Canary Deployment — Quick Reality Summary

* **Definition (real):**
  Canary = **run v1 and v2 together + control how requests are routed between them**

---

### 🔁 How traffic is routed

* **Random (default):** each request randomly goes to v1/v2 (same user can hit both)
* **Sticky (session-based):** a user stays on one version (via cookies/session)
* **Targeted:** specific users (headers, region, internal users) go to v2

---

### ⚖️ Traffic shifting

* **Manual:** you update weights (e.g., 90/10 → 50/50 → 0/100)
* **Automated:** tools like **Argo Rollouts** shift traffic based on metrics

---

### 📈 Scaling behavior

* Traffic split ≠ scaling
* More traffic → more load → **HPA scales pods**

---

### 🧠 Key difference from Rolling Update

* **Rolling Update:** pods change → traffic follows
* **Canary:** traffic changes → pods scale accordingly

---

### 🎯 Purpose

* Test new version on real users
* Reduce risk
* Enable instant rollback (just shift traffic back)

---

### 🔥 Core components

```text
Deployments (v1, v2)
Services
Traffic Router (ALB / Ingress)
Metrics (Prometheus + Grafana)
HPA (scaling)
```

---

### ⚠️ Golden rule

> Canary is NOT about pods — it’s about **traffic control + validation using real metrics**

---

# 🏗️ 2. Core Components (Mental Model)

You must always think in these 4 layers:

```text
1. Deployment → versions (v1, v2)
2. Service → exposes pods
3. Traffic Router → decides traffic split
4. Metrics → decides success/failure
```

---

## 🔥 In AWS EKS:

* Traffic Router → **AWS Load Balancer Controller**
* Metrics → **Prometheus** + **Grafana**
* Scaling → HPA

---

# ⚙️ 3. Full Practical Example (Step-by-Step)

---

## ✅ Step 1: Stable Version (v1)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-v1
spec:
  replicas: 4
  selector:
    matchLabels:
      app: myapp
      version: v1
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
        - name: app
          image: myapp:v1
          ports:
            - containerPort: 8080
```

---

## 🧪 Step 2: Canary Version (v2)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-v2
spec:
  replicas: 1   # start small
  selector:
    matchLabels:
      app: myapp
      version: v2
  template:
    metadata:
      labels:
        app: myapp
        version: v2
    spec:
      containers:
        - name: app
          image: myapp:v2
          ports:
            - containerPort: 8080
```

---

## 🔗 Step 3: Services (IMPORTANT DESIGN)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-v1
spec:
  selector:
    app: myapp
    version: v1
  ports:
    - port: 80
      targetPort: 8080
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-v2
spec:
  selector:
    app: myapp
    version: v2
  ports:
    - port: 80
      targetPort: 8080
```

---

# ⚖️ 4. Traffic Control (CORE OF CANARY)

---

## 🔥 ALB Weighted Routing

```yaml
annotations:
  alb.ingress.kubernetes.io/actions.forward: >
    {
      "Type":"forward",
      "ForwardConfig":{
        "TargetGroups":[
          {
            "ServiceName":"svc-v1",
            "ServicePort":"80",
            "Weight":90
          },
          {
            "ServiceName":"svc-v2",
            "ServicePort":"80",
            "Weight":10
          }
        ]
      }
    }
```

---

## 🧠 Interpretation:

```text
90% users → v1
10% users → v2
```

---

# 🔁 5. Rollout Strategy (REAL FLOW)

This is what companies actually do:

---

## Phase 1: Initial Canary

```text
v2 pods: 1
Traffic: 5–10%
```

Monitor:

* Errors
* Latency
* Logs

---

## Phase 2: Gradual Increase

```text
10% → 25% → 50%
```

At each step:

* Wait (5–30 mins or more)
* Observe metrics

---

## Phase 3: Full Rollout

```text
100% → v2
Delete v1
```

---

# 📊 6. Metrics You MUST Monitor

Using:

* **Prometheus**
* **Grafana**

---

## 🔥 Critical Metrics:

### 1. Error Rate

```text
HTTP 5xx spikes → rollback
```

### 2. Latency

```text
Response time increases → bad release
```

### 3. Throughput

```text
Requests/sec drop → performance issue
```

### 4. Resource Usage

```text
CPU/Memory high → scaling issue
```

---

# 📈 7. Auto Scaling (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-v2-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-v2
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
```

---

## 🧠 Real Behavior:

```text
Traffic ↑ → CPU ↑ → HPA ↑ pods
```

---

# 🔄 8. Rollback Strategy (CRITICAL)

### Immediate rollback:

```text
Set weight:
100% → v1
0% → v2
```

---

### Why Canary is powerful:

* No redeploy needed
* Instant recovery

---

# ⚠️ 9. Common Mistakes (VERY IMPORTANT)

---

## ❌ Mistake 1: Thinking pods = traffic

> Wrong → traffic is controlled by ALB, not pod count

---

## ❌ Mistake 2: No monitoring

> Canary without metrics = useless

---

## ❌ Mistake 3: Large initial traffic

```text
Starting with 50% = risky
```

---

## ❌ Mistake 4: No HPA

> v2 crashes under load

---

# 🧪 10. Advanced Canary Patterns

---

## 🎯 Header-based Canary

```text
If header = "beta-user" → v2
Else → v1
```

---

## 🌍 Region-based Canary

```text
India → v2
US → v1
```

---

## 👨‍💻 Internal Testing

```text
Company employees → v2
Users → v1
```

---

# 🚀 11. CI/CD Integration (VERY IMPORTANT FOR YOU)

Your pipeline should:

---

## Step 1: Deploy v2

* Build Docker image
* Push to ECR
* Apply deployment

---

## Step 2: Set 10% traffic

* Update Ingress

---

## Step 3: Wait + Monitor

---

## Step 4: Auto or manual approval

---

## Step 5: Increase traffic

---

# 🔥 12. Interview-Level Explanation (REMEMBER THIS)

> Canary deployment allows gradual exposure of a new version to real users by controlling traffic percentage, validating system behavior using metrics, and enabling instant rollback without full deployment replacement.

---

# 🧠 Final Mental Model (KEEP THIS FOREVER)

```text
Deployment → creates versions
Service → exposes versions
ALB → controls traffic split
HPA → handles load
Prometheus → decides success/failure
```

---

# 🚀 What YOU should build (for your project)

To make your project stand out:

✅ EKS cluster
✅ ALB Ingress Canary
✅ HPA
✅ Prometheus + Grafana
✅ CI/CD pipeline controlling traffic

---

If you want, next I can convert this into:

👉 **clean markdown (.md) file for your notes**
👉 OR help you **implement this step-by-step on your EKS cluster**

Just tell me 👍
