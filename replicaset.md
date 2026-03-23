# 🔁 Kubernetes ReplicaSet — Complete Notes

---

## 📌 What is a ReplicaSet?

A **ReplicaSet (RS)** ensures a specified number of identical Pods are always running.

> 🎯 Its only job: **Maintain desired replica count**

---

## 🧠 Architecture Position

```
Deployment
    ↓ manages
ReplicaSet
    ↓ maintains
Pods
```

In real-world usage:

* You rarely create ReplicaSet directly.
* Deployment creates & manages ReplicaSets.

---

## 📄 Basic ReplicaSet YAML

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
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

Number of Pods to maintain.

If:

* Pod crashes → RS creates new one
* Pod deleted → RS recreates it
* Node fails → RS reschedules new Pod

---

### `selector` 🔒 (Important)

Defines which Pods this ReplicaSet owns.

Must match:

```
spec.template.metadata.labels
```

⚠️ Overlapping selectors between controllers cause conflicts.

---

### `template`

Pod blueprint.

ReplicaSet uses this template only when:

* Creating new Pods
* Replacing deleted Pods

---

## 🚨 Important Behavior

### ReplicaSet DOES:

* Maintain replica count
* Replace failed Pods
* Self-heal

### ReplicaSet DOES NOT:

* Perform rolling updates
* Support rollback
* Maintain version history
* Detect image changes automatically

---

## 🔥 What Happens If You Change Image?

If you run:

```bash
kubectl set image rs nginx-replicaset nginx=nginx:1.28
```

* Pod template updates
* Existing Pods continue running
* RS sees correct replica count
* Nothing happens

To apply new image, you must:

### Option 1 — Delete Pods

```bash
kubectl delete pod -l app=nginx
```

ReplicaSet recreates them with new image.

### Option 2 — Scale Down & Up

```bash
kubectl scale rs nginx-replicaset --replicas=0
kubectl scale rs nginx-replicaset --replicas=3
```

---

## 🔍 How ReplicaSet Decides to Create/Delete Pods

Logic:

```
If currentPods < replicas → create Pods
If currentPods > replicas → delete Pods
Else → do nothing
```

That’s it.

No intelligence beyond count management.

---

## 📊 Scaling ReplicaSet

```bash
kubectl scale rs nginx-replicaset --replicas=5
```

---

## 🧨 Risk of Using ReplicaSet Directly

If you manually create Pods with matching labels:

```yaml
labels:
  app: nginx
```

ReplicaSet may:

* Adopt them
* Delete them if over replica count

ReplicaSet controls based on selector only.

---

## 🆚 ReplicaSet vs ReplicationController

ReplicaSet is newer and supports:

* `matchLabels`
* `matchExpressions`

ReplicationController is legacy.

---

## 🆚 ReplicaSet vs Deployment

| Feature                     | ReplicaSet | Deployment |
| --------------------------- | ---------- | ---------- |
| Maintain replica count      | ✅          | ✅          |
| Rolling updates             | ❌          | ✅          |
| Rollback                    | ❌          | ✅          |
| Version history             | ❌          | ✅          |
| Used directly in production | Rare       | Yes        |

---

## 🎯 Interview Summary

* ReplicaSet ensures N Pods are always running.
* It only manages quantity, not versions.
* Image changes don’t restart Pods automatically.
* Deployment adds rolling update & rollback on top of ReplicaSet.

---

If you want next:

* 🔥 How ReplicaSet adopts orphan Pods
* 🔥 What ownerReferences means
* 🔥 Internal reconciliation loop explanation
