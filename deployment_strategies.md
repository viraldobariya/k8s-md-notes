# 🚀 Kubernetes Deployment Strategies (Practical DevOps Notes)

---

# 🧠 1. Why Deployment Strategies Matter

When you release a new version of your app:

* You must **avoid downtime**
* You must **minimize risk**
* You must **be able to rollback**

👉 Deployment strategies define *how traffic shifts from old version → new version*

---

# 🟢 2. Rolling Update (Default Kubernetes Strategy)

## 💡 Idea

Gradually replace old pods with new pods.

---

## ⚙️ Configuration

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

---

## 🔍 Meaning

* `maxUnavailable: 1` → only 1 pod can go down
* `maxSurge: 1` → 1 extra pod can be created temporarily

---

## 🧩 Example

Initial state:

```
3 replicas (v1)
Pod-1 v1
Pod-2 v1
Pod-3 v1
```

### Step-by-step rollout:

1. Add new pod (v2)

```
Pod-1 v1
Pod-2 v1
Pod-3 v1
Pod-4 v2
```

2. Remove old pod

```
Pod-2 v1
Pod-3 v1
Pod-4 v2
```

3. Repeat until all v2

```
Pod-4 v2
Pod-5 v2
Pod-6 v2
```

---

## ✅ Pros

* No downtime
* Simple (default)
* No extra setup

---

## ❌ Cons

* No traffic control (users randomly hit v1/v2)
* Harder to detect issues early

---

## 🧠 Real Use

* Standard deployments
* Most backend APIs

---

# 🔵 3. Blue-Green Deployment

## 💡 Idea

Run two environments → switch traffic instantly

* Blue = current (v1)
* Green = new (v2)

---

## 🧩 Flow

### Step 1: Current state

```
v1 (Blue)
Pod-1
Pod-2
Pod-3
```

---

### Step 2: Deploy v2 separately

```
v1 (Blue)        v2 (Green)
Pod-1            Pod-4
Pod-2            Pod-5
Pod-3            Pod-6
```

👉 No traffic to v2 yet

---

### Step 3: Switch traffic

Update Service:

```yaml
selector:
  app: v2
```

👉 Now all traffic → v2

---

### Step 4: Remove v1

---

## ✅ Pros

* Instant rollback (switch back)
* Full testing before release
* Zero downtime

---

## ❌ Cons

* Double infrastructure cost
* Sudden traffic switch (risk spike)

---

## 🧠 Real Use

* Critical production apps
* When rollback must be instant

---

## ⚙️ Kubernetes Setup

* Two Deployments:

  * `app-v1`
  * `app-v2`
* One Service → switch selector

---

# 🟡 4. Canary Deployment

## 💡 Idea

Release new version to small % of users → increase gradually

---

## 🧩 Flow

### Step 1: Start

```
All pods → v1
```

---

### Step 2: Add small v2

```
3 pods v1
1 pod v2
```

👉 ~25% traffic → v2

---

### Step 3: Observe

* Errors
* Logs
* Metrics

---

### Step 4: Increase gradually

```
2 pods v1
2 pods v2
```

Then:

```
All → v2
```

---

## ✅ Pros

* Safest deployment
* Real user testing
* Easy to stop

---

## ❌ Cons

* Complex setup
* Needs monitoring + traffic control

---

## 🧠 Real Use

* High-scale apps
* User-facing features
* Risky changes

---

## ⚙️ Kubernetes Implementation

### Option 1: Simple (Replica-based)

* Manually scale v1 and v2 deployments

---

### Option 2: Advanced (Traffic Split)

Using:

* Ingress Controller (NGINX)
* Service Mesh (Istio / Linkerd)

👉 Example (NGINX Canary):

```yaml
nginx.ingress.kubernetes.io/canary: "true"
nginx.ingress.kubernetes.io/canary-weight: "20"
```

👉 20% traffic → v2

---

# 🔥 5. Comparison Table

| Feature          | Rolling Update  | Blue-Green     | Canary          |
| ---------------- | --------------- | -------------- | --------------- |
| Traffic Control  | ❌ No            | ❌ No           | ✅ Yes           |
| Deployment Style | Gradual replace | Instant switch | Gradual traffic |
| Downtime         | None            | None           | None            |
| Risk             | Medium          | Medium         | Low             |
| Rollback         | Medium          | Instant        | Easy            |
| Infra Cost       | Normal          | High           | Medium          |
| Complexity       | Easy            | Medium         | High            |

---

# 🧠 6. Real DevOps Decision Guide

### Use Rolling Update when:

* Normal deployments
* Backend APIs
* No need for traffic control

---

### Use Blue-Green when:

* Need instant rollback
* High-risk release
* Can afford double infra

---

### Use Canary when:

* Large user base
* Want safe gradual rollout
* Have monitoring (Prometheus, Grafana)

---

# ⚠️ 7. Important Production Concepts

## 🔍 Readiness Probe (VERY IMPORTANT)

* Ensures pod is ready before receiving traffic
* Prevents bad deployments

---

## 🔁 Rollback

```bash
kubectl rollout undo deployment <name>
```

---

## 📊 Monitoring Required for Canary

* Error rate
* Latency
* CPU/memory

---

## 🔐 Best Practice

* Never deploy without:

  * Health checks
  * Monitoring
  * Rollback plan

---

# 💥 8. Simple Memory Trick

* Rolling → Replace slowly
* Blue-Green → Switch instantly
* Canary → Test gradually

---

# 🚀 9. Industry Reality

* 70% → Rolling Update
* 20% → Canary
* 10% → Blue-Green

👉 But:

* Big companies → Canary + Service Mesh
* Startups → Rolling Update

---

# 🧠 Final Thought

> Deployment is not just “release code”
> It’s about **controlling risk in production**

---
