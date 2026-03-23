# 🟣 Vertical Pod Autoscaler (VPA) — Complete In-Depth Notes (Clear + Practical)

> **VPA automatically adjusts CPU and Memory requests/limits of Pods.**

If:

* HPA → changes **number of Pods**
* VPA → changes **size of each Pod**

---

# 1️⃣ Why VPA is Needed?

Imagine your app:

* Uses more memory than expected → OOMKilled ❌
* Uses too much CPU request → wasting cluster resources ❌
* You guessed wrong resource values ❌

VPA solves this by:

```id="uwlps2"
Monitoring usage
Recommending optimal resources
Automatically updating Pod resources
```

---

# 2️⃣ What VPA Scales

VPA modifies:

```id="n6v5au"
resources.requests
resources.limits
```

It does NOT change:

* Number of replicas
* Nodes

---

# 3️⃣ How VPA Works Internally

Architecture:

```text id="ygz5h4"
Metrics Server
     ↓
VPA Recommender
     ↓
VPA Updater
     ↓
Evict Pod
     ↓
New Pod with updated resources
```

Important:

⚠ VPA may **restart Pods** to apply new resource values.

---

# 4️⃣ VPA Components

VPA has 3 main parts:

| Component            | Role                       |
| -------------------- | -------------------------- |
| Recommender          | Calculates ideal resources |
| Updater              | Evicts Pod if needed       |
| Admission Controller | Mutates Pod on creation    |

---

# 5️⃣ Basic VPA Example

## Step 1 — Deployment

```yaml id="sbjv7q"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: nginx
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
```

---

## Step 2 — Create VPA

```yaml id="a8q3zy"
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: api-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: api
  updatePolicy:
    updateMode: "Auto"
```

---

# 6️⃣ What Happens?

VPA monitors usage.

If actual usage is:

```id="0xv50o"
CPU: 400m
Memory: 600Mi
```

VPA updates Pod spec:

```id="0zld8k"
cpu: 450m
memory: 700Mi
```

Pods are restarted with new values.

---

# 7️⃣ VPA Update Modes (Very Important)

| Mode     | Behavior                         |
| -------- | -------------------------------- |
| Off      | Only recommendations             |
| Initial  | Applies only at Pod creation     |
| Auto     | Automatically updates & restarts |
| Recreate | Similar to Auto (older mode)     |

---

## Recommendation-Only Mode (Safe for Production Start)

```yaml id="4j9zux"
updatePolicy:
  updateMode: "Off"
```

Check recommendation:

```bash id="p36xcr"
kubectl describe vpa api-vpa
```

---

# 8️⃣ VPA vs HPA (Clear Difference)

| Feature                 | HPA            | VPA             |
| ----------------------- | -------------- | --------------- |
| Changes replicas        | ✅              | ❌               |
| Changes CPU/Memory size | ❌              | ✅               |
| Restarts Pods           | ❌              | ✅               |
| Best for                | Stateless apps | Resource tuning |

---

# 9️⃣ Can We Use HPA + VPA Together?

Generally:

❌ Not recommended on same resource metric (CPU).

Why?

* HPA scales based on CPU usage %
* VPA changes CPU request
* Both may conflict

However:

✔ Can use HPA (CPU) + VPA (Memory) carefully
✔ Or HPA for scaling + VPA in recommendation mode

---

# 🔟 Real Production Example

Imagine backend API connecting to:

* PostgreSQL

Traffic stable, but memory usage unpredictable.

Instead of manually tuning:

VPA automatically adjusts memory:

```id="evc50n"
128Mi → 512Mi → 768Mi
```

Prevents:

```id="jq2qpj"
OOMKilled errors
```

---

# 1️⃣1️⃣ When NOT to Use VPA

❌ Large StatefulSets like:

* MySQL
* Apache Kafka

Because restart = downtime risk.

❌ Latency-sensitive apps

❌ Apps that cannot restart safely

---

# 1️⃣2️⃣ Good Use Cases

✔ Batch workloads
✔ Internal APIs
✔ Memory unpredictable apps
✔ Cost optimization

---

# 1️⃣3️⃣ Important Commands

Install VPA (if not installed):

```bash id="o4zwdx"
kubectl apply -f https://github.com/kubernetes/autoscaler/vertical-pod-autoscaler/...
```

Check VPA:

```bash id="k5r3ml"
kubectl get vpa
```

Describe:

```bash id="n4h5zv"
kubectl describe vpa api-vpa
```

---

# 1️⃣4️⃣ Resource Recommendation Output

When you describe VPA:

```id="0zz3bf"
Recommendations:
  Target:
    cpu: 350m
    memory: 600Mi
  LowerBound:
  UpperBound:
```

You can use this manually even if Auto mode is off.

---

# 1️⃣5️⃣ Internal Flow (Visualization)

```text id="ghzv23"
Pod running
   ↓
Metrics collected
   ↓
Recommender calculates optimal values
   ↓
Updater evicts Pod
   ↓
New Pod created with new resources
```

---

# 1️⃣6️⃣ HPA + VPA + Cluster Autoscaler (Big Picture)

```text id="l49gzt"
HPA → scales Pods
VPA → scales Pod size
Cluster Autoscaler → scales Nodes
```

All three together create:

```id="vb6r9n"
Fully elastic Kubernetes cluster
```

---

# 🧠 Final Understanding

HPA:

> More Pods

VPA:

> Bigger/Smaller Pods

Cluster Autoscaler:

> More/Fewer Nodes

---

# 🎯 One-Line Summary

> VPA automatically tunes CPU & Memory requests for Pods based on real usage.

---

If you want next:

* 🔥 HPA vs VPA deep interview questions
* 🔥 Real-world scaling strategy design
* 🔥 Cluster Autoscaler full deep notes
* 🔥 Production autoscaling best practices
