# 🔄 Rolling Update (Kubernetes)

A **Rolling Update** is a deployment strategy in **Kubernetes** where Pods are **updated gradually**, without stopping the entire application.

It replaces old Pods with new ones **in small batches**, ensuring:

* ✅ Zero (or minimal) downtime
* ✅ Continuous availability
* ✅ Controlled upgrade

---

## 📌 Where It Is Used?

Rolling updates are mainly used with:

* Kubernetes **Deployment**
* Also supported by StatefulSets (with ordered updates)

---

## 🧠 What Actually Happens?

Suppose:

```yaml
replicas: 3
image: nginx:1.25
```

You change image to:

```yaml
image: nginx:1.26
```

When you run:

```bash
kubectl apply -f deployment.yaml
```

### Kubernetes will:

1. Create **1 new Pod** with `nginx:1.26`
2. Wait until it becomes **Ready**
3. Delete **1 old Pod** (`nginx:1.25`)
4. Repeat until all Pods are updated

Old and new versions run **simultaneously** during transition.

---

## ⚙️ How It Is Controlled

Inside Deployment spec:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

### 🔹 `maxUnavailable`

* How many Pods can be unavailable during update
* Example: `1` → At least 2 Pods always running (if replicas=3)

### 🔹 `maxSurge`

* Extra Pods allowed above desired replicas
* Example: `1` → Can temporarily run 4 Pods during update

---

## 📊 Example Scenario (replicas = 3)

| Step   | Running Old | Running New | Total |
| ------ | ----------- | ----------- | ----- |
| Start  | 3           | 0           | 3     |
| Step 1 | 3           | 1           | 4     |
| Step 2 | 2           | 1           | 3     |
| Step 3 | 2           | 2           | 4     |
| Step 4 | 1           | 2           | 3     |
| End    | 0           | 3           | 3     |

---

## 🚀 Why Rolling Update Is Important

* No downtime
* Safe production upgrades
* Easy rollback (`kubectl rollout undo`)
* Default strategy in Kubernetes Deployments

---

## 🔁 Difference from Recreate Strategy

| RollingUpdate       | Recreate               |
| ------------------- | ---------------------- |
| Gradual replacement | Deletes all Pods first |
| No downtime         | Causes downtime        |
| Default strategy    | Must be configured     |

---

# 🎯 In Simple Words

> Rolling update = "Update the app slowly, not all at once."

---

If you want, I can also explain how this works internally with ReplicaSets (since you’re learning K8s deeply).
