# 🐳 Kubernetes DaemonSet — Complete Notes

---

## 📌 What is a DaemonSet?

A **DaemonSet** ensures:

> 🎯 **One Pod runs on every eligible Node**

Unlike Deployment (fixed replica count),
DaemonSet = **Node-based scheduling**.

---

## 🧠 Where It’s Used

Common system-level workloads:

* Log collectors (Fluentd)
* Monitoring agents (Node exporter)
* Security agents
* CNI plugins
* Storage daemons

---

## 🏗 Architecture Idea

```
If 5 nodes → 5 Pods
If new node joins → 1 new Pod automatically
If node removed → Pod removed
```

Pods are tied to Nodes.

---

## 📄 Basic DaemonSet YAML

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemonset
  namespace: nginx
spec:
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
          image: nginx:latest
          ports:
            - containerPort: 80
```

---

## 🔹 Key Characteristics

### ✅ No `replicas` Field

Replica count is determined by:

```
Number of eligible nodes
```

---

### ✅ Automatically Runs on New Nodes

When a new node joins cluster:

* DaemonSet immediately schedules a Pod there.

---

### ✅ One Pod Per Node

By default:

* Exactly 1 Pod per Node.

(Unless using advanced scheduling tricks.)

---

## 🔄 Update Strategy

Default:

```yaml
strategy:
  type: RollingUpdate
```

During update:

* Updates Pods node by node.
* Controlled by:

```yaml
rollingUpdate:
  maxUnavailable: 1
```

---

## 🎯 Difference from Deployment

| Feature              | Deployment | DaemonSet            |
| -------------------- | ---------- | -------------------- |
| Fixed replicas       | ✅          | ❌                    |
| One per node         | ❌          | ✅                    |
| Used for apps        | ✅          | ❌ (system workloads) |
| Auto add on new node | ❌          | ✅                    |

---

## 🧠 Scheduling Behavior

DaemonSet ignores normal scheduling logic:

* Automatically targets all nodes.
* Can use:

```yaml
nodeSelector
tolerations
affinity
```

To restrict which nodes get the Pod.

---

## 🔥 Example: Run Only on Worker Nodes

```yaml
spec:
  template:
    spec:
      nodeSelector:
        node-type: worker
```

---

## 🚨 Important Notes

* Selector is immutable (like Deployment).
* Changing Pod template triggers rollout.
* Often runs in `kube-system` namespace.

---

## 🎯 Real-World Example

If you install:

* Monitoring stack
* Logging stack
* Security scanning

They use DaemonSet to ensure:

```
Every node has the agent running.
```

---

## 🧠 Mental Model

```
Deployment → Scale by replicas
DaemonSet → Scale by nodes
```

---

## 🎯 Interview One-Liner

> A DaemonSet ensures one Pod runs on every eligible node, commonly used for cluster-level agents like logging or monitoring.

---

If you want next:

* 🔥 DaemonSet vs StatefulSet
* 🔥 How DaemonSet handles taints
* 🔥 Real production architecture usage
