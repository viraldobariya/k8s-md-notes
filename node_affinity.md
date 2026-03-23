# 🟢 Node Affinity — Complete In-Depth Notes (Clear + Practical)

> **Node Affinity** controls which **Nodes a Pod can be scheduled on**, based on node labels.

If:

* Taints → repel Pods
* NodeSelector → simple matching
* **Node Affinity → advanced scheduling rules**

---

# 1️⃣ Why Node Affinity is Needed?

In real clusters, nodes may be:

* SSD storage nodes
* GPU nodes
* High-memory nodes
* Production-only nodes
* Different zones (zone-a, zone-b)

You may want:

```text
Backend Pods → only high-memory nodes
ML Pods → only GPU nodes
Database → only SSD nodes
```

Node Affinity solves this.

---

# 2️⃣ Node Labels (Foundation)

Node Affinity works on **Node labels**.

Check node labels:

```bash
kubectl get nodes --show-labels
```

Add label:

```bash
kubectl label nodes node1 disktype=ssd
```

Now node1 has:

```
disktype=ssd
```

---

# 3️⃣ NodeSelector vs Node Affinity

### NodeSelector (Simple)

```yaml
nodeSelector:
  disktype: ssd
```

Only exact match. No flexibility.

---

### Node Affinity (Advanced)

Supports:

* Multiple conditions
* AND/OR logic
* Required vs Preferred rules
* Operators (In, NotIn, Exists, etc.)

---

# 4️⃣ Types of Node Affinity

There are 2 types:

| Type                                            | Meaning          |
| ----------------------------------------------- | ---------------- |
| requiredDuringSchedulingIgnoredDuringExecution  | Hard requirement |
| preferredDuringSchedulingIgnoredDuringExecution | Soft preference  |

---

# 5️⃣ Required Node Affinity (Hard Rule)

If rule not satisfied → Pod will NOT schedule.

---

## Example — Require SSD Node

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-ssd
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd
  containers:
    - name: nginx
      image: nginx
```

Meaning:

```
Only schedule on nodes where:
disktype=ssd
```

If no such node → Pod remains Pending.

---

# 6️⃣ Preferred Node Affinity (Soft Rule)

Scheduler tries to place Pod there.

If not possible → schedules elsewhere.

---

## Example

```yaml
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```

Meaning:

```
Prefer SSD node
But if not available → schedule anywhere
```

---

# 7️⃣ Operators in Node Affinity

| Operator     | Meaning            |
| ------------ | ------------------ |
| In           | Value must match   |
| NotIn        | Must not match     |
| Exists       | Key must exist     |
| DoesNotExist | Key must not exist |
| Gt           | Greater than       |
| Lt           | Less than          |

---

## Example — Exists

```yaml
matchExpressions:
  - key: gpu
    operator: Exists
```

Means:

```
Schedule only on nodes having label gpu
```

---

# 8️⃣ Real Production Example

Imagine:

You have database Pods running:

* PostgreSQL

You want DB only on:

```
high-memory=true
```

### Step 1 — Label Node

```bash
kubectl label nodes node2 high-memory=true
```

---

### Step 2 — StatefulSet with Node Affinity

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: high-memory
              operator: In
              values:
                - "true"
```

Now DB runs only on high-memory nodes.

---

# 9️⃣ AND / OR Logic Explained

### Inside matchExpressions → AND

```yaml
matchExpressions:
  - key: disktype
    operator: In
    values: [ssd]
  - key: zone
    operator: In
    values: [zone-a]
```

Means:

```
disktype=ssd AND zone=zone-a
```

---

### Multiple nodeSelectorTerms → OR

```yaml
nodeSelectorTerms:
  - matchExpressions:
      - key: zone
        operator: In
        values: [zone-a]
  - matchExpressions:
      - key: zone
        operator: In
        values: [zone-b]
```

Means:

```
zone-a OR zone-b
```

---

# 🔟 Scheduling Flow

```text
Pod created
   ↓
Scheduler checks node labels
   ↓
Affinity rules satisfied?
   ↓
Yes → Schedule
No → Skip node
```

---

# 1️⃣1️⃣ IgnoredDuringExecution Meaning

Important part:

```
requiredDuringSchedulingIgnoredDuringExecution
```

Means:

* Rule checked only at scheduling time
* If node label changes later → Pod NOT evicted

---

# 1️⃣2️⃣ Node Affinity vs Taints (Clear Difference)

| Feature    | Node Affinity | Taints     |
| ---------- | ------------- | ---------- |
| Applied to | Pod           | Node       |
| Purpose    | Attract Pod   | Repel Pod  |
| Direction  | Pod → Node    | Node → Pod |

Think:

* Affinity = "I want this node"
* Taint = "You cannot come here"

---

# 1️⃣3️⃣ Real-World Use Cases

### 1️⃣ GPU Workloads

```yaml
matchExpressions:
  - key: gpu
    operator: Exists
```

ML workloads only on GPU nodes.

---

### 2️⃣ Zone-Aware Deployment

```
topology.kubernetes.io/zone=zone-a
```

Used in multi-zone clusters.

---

### 3️⃣ Dedicated DB Nodes

For systems like:

* MySQL
* MongoDB

Keep DB isolated from application nodes.

---

# 1️⃣4️⃣ Common Mistakes

❌ Wrong label key
❌ Using required when preferred needed
❌ Forgetting to label nodes
❌ Confusing with podAffinity

---

# 1️⃣5️⃣ Complete Example (Deployment + Affinity)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: disktype
                    operator: In
                    values:
                      - ssd
      containers:
        - name: backend
          image: nginx
```

---

# 🧠 Final Understanding

Node Affinity:

> Advanced rule to control where Pods run.

Required:

> Hard rule

Preferred:

> Soft rule

Used for:

* Performance
* Isolation
* Resource optimization
* Compliance

---

# 🎯 One-Line Summary

> Node Affinity lets you control Pod placement using node labels with advanced matching rules.

---

If you want next:

* 🔥 Pod Affinity & Anti-Affinity notes
* 🔥 Scheduler internal decision flow
* 🔥 Real production scheduling strategy design
* 🔥 Topology spread constraints explanation
