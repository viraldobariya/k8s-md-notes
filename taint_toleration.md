# 🟣 Taints & Tolerations in Kubernetes — Complete In-Depth Notes (Clear + Practical)

> **Taints** are applied to **Nodes** to repel Pods.
> **Tolerations** are applied to **Pods** to allow scheduling onto tainted Nodes.

Think of it like:

```
Node says: "Do not schedule Pods here ❌"
Pod says: "I can tolerate this taint ✅"
```

---

# 1️⃣ Why Do We Need Taints?

In real clusters, some nodes are special:

* GPU nodes
* High-memory nodes
* Dedicated DB nodes
* Production-only nodes

You don’t want normal Pods to run there accidentally.

So we use **taints** to protect nodes.

---

# 2️⃣ What is a Taint?

A taint is applied to a **Node**.

### Syntax:

```bash
kubectl taint nodes <node-name> key=value:effect
```

Example:

```bash
kubectl taint nodes node1 env=prod:NoSchedule
```

Now:

```
node1 rejects Pods ❌
```

Unless they tolerate it.

---

# 3️⃣ Taint Structure (Very Important)

```
key = value : effect
```

Example:

```
env = prod : NoSchedule
```

---

# 4️⃣ Taint Effects (3 Types)

| Effect           | Meaning                               |
| ---------------- | ------------------------------------- |
| NoSchedule       | New Pods will NOT be scheduled        |
| PreferNoSchedule | Try to avoid scheduling               |
| NoExecute        | Remove existing Pods + block new ones |

---

## 4.1 NoSchedule

Strict blocking.

Pods without matching toleration:

```
Will not schedule ❌
```

---

## 4.2 PreferNoSchedule

Soft rule.

Scheduler:

```
Tries to avoid node, but may schedule if needed.
```

---

## 4.3 NoExecute

Most powerful.

* Blocks new Pods
* Evicts existing Pods

Used for:

* Node maintenance
* Node failures

---

# 5️⃣ What is a Toleration?

Toleration is added inside Pod spec.

It allows Pod to run on tainted node.

---

## Example — Pod with Toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  tolerations:
    - key: "env"
      operator: "Equal"
      value: "prod"
      effect: "NoSchedule"
  containers:
    - name: nginx
      image: nginx
```

Now this Pod:

```
Can run on env=prod:NoSchedule node
```

---

# 6️⃣ Real Example Scenario (Clear)

### Step 1 — Taint Node

```bash
kubectl taint nodes node1 dedicated=db:NoSchedule
```

Now:

```
node1 is reserved for DB workloads
```

---

### Step 2 — Normal Pod (Without Toleration)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: normal-app
spec:
  containers:
    - name: nginx
      image: nginx
```

Result:

```
Will NOT schedule on node1 ❌
```

---

### Step 3 — DB Pod (With Toleration)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db-pod
spec:
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "db"
      effect: "NoSchedule"
  containers:
    - name: mysql
      image: mysql:8
```

Now:

```
Can schedule on node1 ✅
```

Example DB:

* MySQL
* PostgreSQL

---

# 7️⃣ Operator Types in Toleration

| Operator | Meaning              |
| -------- | -------------------- |
| Equal    | key=value must match |
| Exists   | Only key must match  |

---

## Exists Example

If node taint:

```
gpu=true:NoSchedule
```

Pod can tolerate with:

```yaml
tolerations:
  - key: "gpu"
    operator: "Exists"
    effect: "NoSchedule"
```

Value not required.

---

# 8️⃣ NoExecute + TolerationSeconds

Used for eviction timing.

Example:

```yaml
tolerations:
  - key: "node.kubernetes.io/not-ready"
    operator: "Exists"
    effect: "NoExecute"
    tolerationSeconds: 60
```

Meaning:

* If node becomes NotReady
* Pod stays for 60 seconds
* Then gets evicted

---

# 9️⃣ Default Taints in Kubernetes

Kubernetes automatically taints:

Control-plane nodes:

```
node-role.kubernetes.io/control-plane:NoSchedule
```

This prevents normal workloads from running on master.

Unless tolerated.

---

# 🔟 Taints vs NodeSelector (Important Difference)

| Feature    | Taints           | NodeSelector     |
| ---------- | ---------------- | ---------------- |
| Applied to | Node             | Pod              |
| Purpose    | Repel pods       | Attract pods     |
| Direction  | Block scheduling | Force scheduling |

Think:

* NodeSelector → "I want this node"
* Taint → "You cannot come here"

---

# 1️⃣1️⃣ Real Production Use Cases

### 1️⃣ GPU Nodes

Taint GPU node:

```
gpu=true:NoSchedule
```

Only ML workloads tolerate it.

---

### 2️⃣ Dedicated Database Nodes

```
dedicated=db:NoSchedule
```

Only DB StatefulSets allowed.

---

### 3️⃣ Spot/Preemptible Nodes

Mark as:

```
spot=true:PreferNoSchedule
```

Non-critical workloads use them.

---

# 1️⃣2️⃣ Scheduling Flow (Visualization)

```text
Pod created
   ↓
Scheduler checks node
   ↓
Node has taint?
   ↓
Yes → Does Pod tolerate?
   ↓
No → Reject
Yes → Schedule
```

---

# 1️⃣3️⃣ Common Mistakes

❌ Adding toleration but no taint
❌ Wrong effect mismatch
❌ Forgetting operator field
❌ Confusing with nodeSelector
❌ Using NoExecute accidentally

---

# 1️⃣4️⃣ Important Commands

Check taints:

```bash
kubectl describe node node1
```

Remove taint:

```bash
kubectl taint nodes node1 dedicated=db:NoSchedule-
```

Notice trailing `-` removes taint.

---

# 🧠 Final Understanding (Very Clear)

Taint:

> Node-level restriction

Toleration:

> Pod-level permission

They work together to:

* Control scheduling
* Isolate workloads
* Protect special nodes
* Manage infrastructure efficiently

---

# 🎯 One-Line Summary

> Taints repel Pods.
> Tolerations allow Pods to bypass that restriction.

---

If you want next:

* 🔥 Node Affinity vs Taints deep comparison
* 🔥 Real interview questions on scheduling
* 🔥 Scheduler internal decision flow
* 🔥 Production architecture scheduling strategy
