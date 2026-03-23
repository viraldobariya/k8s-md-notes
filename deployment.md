# 🚀 Kubernetes Deployment — Complete Notes

---

## 📌 What is a Deployment?

A **Deployment** is a higher-level Kubernetes controller that:

* Manages ReplicaSets
* Performs rolling updates
* Enables rollback
* Maintains desired state
* Provides zero-downtime upgrades

---

## 🧠 Architecture

```
Deployment
    ↓ manages
ReplicaSet
    ↓ maintains
Pods
```

You never manage ReplicaSet directly in production.

---

## 📄 Basic Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: nginx
spec:
  replicas: 3

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

## 🔹 Key Fields Explained

### `replicas`

Number of Pod instances to run.

---

### `selector` 🔒 (Immutable)

Defines which Pods this Deployment owns.

Must match:

```
spec.template.metadata.labels
```

Cannot be changed after creation.

---

### `template`

Pod blueprint.
Defines:

* Labels
* Containers
* Image
* Resources
* Probes
* Volumes

Any change here triggers a rollout.

---

## 🔄 Rolling Update (Default Strategy)

When image changes:

```yaml
image: nginx:1.28
```

Kubernetes:

1. Creates new ReplicaSet
2. Gradually scales up new Pods
3. Gradually scales down old Pods

---

## ⚙️ Rolling Update Strategy

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```

### `maxSurge`

Extra Pods allowed above desired replicas.

### `maxUnavailable`

Pods allowed to be unavailable during update.

Example:
Replicas = 3

* maxSurge: 1 → can run 4 Pods temporarily
* maxUnavailable: 1 → at least 2 must stay running

---

## 🔁 Rollback

View rollout history:

```bash
kubectl rollout history deployment nginx-deployment
```

Rollback:

```bash
kubectl rollout undo deployment nginx-deployment
```

---

## 📊 Scaling

Manual scaling:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Auto-scaling (HPA) also works with Deployment.

---

## 🛑 What You Cannot Change

* `spec.selector` ❌ (immutable)

Changing it requires deleting & recreating Deployment.

---

## 🔥 When Does New ReplicaSet Get Created?

Whenever this changes:

```
spec.template
```

Examples:

* Image change
* Label change
* Env vars
* Resources
* Probes

Kubernetes calculates a hash of PodTemplate to detect changes.

---

## 🧩 Common Commands

Create:

```bash
kubectl apply -f deployment.yaml
```

Check status:

```bash
kubectl get deploy -n nginx
```

Describe:

```bash
kubectl describe deployment nginx-deployment -n nginx
```

Watch rollout:

```bash
kubectl rollout status deployment nginx-deployment
```

---

## 🆚 Deployment vs ReplicaSet

| Feature           | ReplicaSet | Deployment |
| ----------------- | ---------- | ---------- |
| Maintain replicas | ✅          | ✅          |
| Rolling updates   | ❌          | ✅          |
| Rollback          | ❌          | ✅          |
| Version history   | ❌          | ✅          |
| Production usage  | Rare       | Standard   |

---

## 🎯 Interview Summary

* Deployment manages ReplicaSets.
* ReplicaSet manages Pods.
* Any PodTemplate change creates a new ReplicaSet.
* Selector is immutable.
* Enables safe rolling updates & rollback.

---

If you want next:

* 🔥 Deployment + Service interaction
* 🔥 Blue-Green vs Rolling updates
* 🔥 How HPA works with Deployment
