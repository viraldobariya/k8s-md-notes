# 🟢 PersistentVolumeClaim (PVC) — Complete Long-Term Notes

This is the **most important piece developers use** in Kubernetes storage.
We’ll go **deep, clear, and practical** — no confusion.

---

# 1️⃣ What is a PVC?

A **PersistentVolumeClaim (PVC)** is:

> A request for storage made by a user/application.

It does NOT store data itself.
It is a **bridge between Pod and storage (PV)**.

---

## 🔎 Scope

| Resource | Scope              |
| -------- | ------------------ |
| PV       | 🌍 Cluster-level   |
| PVC      | 📦 Namespace-level |
| Pod      | 📦 Namespace-level |

---

# 2️⃣ Why PVC Exists

Without PVC:

* Pods would need to know storage details (disk, path, cloud, etc.) ❌
* Tight coupling ❌

With PVC:

> Pod only asks: “Give me storage”
> Kubernetes handles the rest ✅

---

# 3️⃣ Architecture Flow (Cluster-Level)

```id="9i4h6h"
Developer creates PVC
        ↓
Kubernetes checks:
   → Existing PV (static)
   → OR StorageClass (dynamic)
        ↓
PV is selected or created
        ↓
PVC binds to PV (1:1)
        ↓
Pod uses PVC
        ↓
Kubelet mounts volume on node
```

---

# 4️⃣ Basic PVC Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
  namespace: nginx
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

---

# 🔎 Key Fields Explained

---

## 🔹 resources.requests.storage

```yaml
requests:
  storage: 5Gi
```

* Minimum storage required
* Kubernetes finds PV ≥ this size

⚠ Important:

* PVC requesting 5Gi can bind to 10Gi PV
* But entire PV is reserved

---

## 🔹 accessModes

Same as PV:

| Mode | Meaning               |
| ---- | --------------------- |
| RWO  | One node read/write   |
| ROX  | Many nodes read-only  |
| RWX  | Many nodes read/write |
| RWOP | One pod only          |

Must match PV.

---

## 🔹 storageClassName

```yaml
storageClassName: gp3
```

Controls provisioning behavior.

### Cases:

| Value   | Meaning                  |
| ------- | ------------------------ |
| `gp3`   | Use this StorageClass    |
| `""`    | Only static PV           |
| Not set | Use default StorageClass |

---

## 🔹 volumeMode

```yaml
volumeMode: Filesystem
```

* Filesystem (default)
* Block (raw disk)

---

# 5️⃣ Binding Process (Very Important)

PVC binds to PV if:

* storageClassName matches
* accessModes compatible
* PV capacity ≥ PVC request
* volumeMode matches

---

## 🔥 Binding Rule

```id="fx4vlc"
1 PVC ↔ 1 PV
```

* No sharing (unless RWX backend supports multiple Pods)

---

# 6️⃣ Static vs Dynamic PVC

---

## 🔹 Static Binding

* Admin creates PV
* Developer creates PVC

PVC waits until matching PV is found.

---

## 🔹 Dynamic Provisioning

PVC triggers PV creation automatically.

```yaml
storageClassName: gp3
```

Flow:

```id="1lajdi"
PVC → StorageClass → Provisioner → PV → Bind
```

---

# 7️⃣ PVC Lifecycle States

| Status  | Meaning            |
| ------- | ------------------ |
| Pending | Waiting for PV     |
| Bound   | Connected to PV    |
| Lost    | PV lost or deleted |

---

# 8️⃣ How Pod Uses PVC

---

## 🔹 Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 1
  template:
    spec:
      containers:
        - name: app
          image: nginx
          volumeMounts:
            - mountPath: /data
              name: storage
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: my-pvc
```

---

## 🔎 What Happens Internally

```id="l1g5iy"
Pod scheduled to node
        ↓
Kubelet sees PVC
        ↓
Finds bound PV
        ↓
Mounts storage to node
        ↓
Mounts inside container at /data
```

---

# 9️⃣ Multi-Replica Behavior (Very Important)

---

## 🔹 RWO (ReadWriteOnce)

* Only one node can mount volume
* Multiple Pods allowed only on SAME node

👉 Deployment with replicas >1 may fail or schedule on same node

---

## 🔹 RWX (ReadWriteMany)

* Multiple nodes can mount
* Used for shared storage (NFS, Ceph)

---

# 🔟 What Happens in Different Scenarios

---

## ✅ Pod Deleted

* PVC remains
* PV remains
* Data persists

---

## ⚠ PVC Deleted

Depends on PV reclaim policy:

| Policy  | Result              |
| ------- | ------------------- |
| Retain  | PV stays (Released) |
| Delete  | PV deleted          |
| Recycle | Deprecated          |

---

## ❌ PV Deleted

* PVC becomes `Lost`
* Pod cannot access storage

---

# 1️⃣1️⃣ Real Storage Behavior

---

## 🔹 Local / hostPath

* No real quota enforcement
* PVC size is logical

---

## 🔹 Cloud (EBS, GCE)

* Real disk created
* Size enforced
* Cannot exceed capacity

---

## 🔹 NFS

* Depends on server quota

---

# 1️⃣2️⃣ Advanced Features

---

## 🔹 Volume Expansion

If enabled in StorageClass:

```yaml
allowVolumeExpansion: true
```

You can increase PVC size.

---

## 🔹 Selector (Rare)

PVC can match specific PV labels:

```yaml
selector:
  matchLabels:
    type: fast
```

---

# 1️⃣3️⃣ Common Mistakes

❌ Thinking PVC creates storage always
❌ Forgetting storageClassName mismatch
❌ Expecting PV sharing automatically
❌ Using RWO with multi-node replicas

---

# 1️⃣4️⃣ Debugging PVC

---

## 🔹 Check PVC

```bash
kubectl get pvc -n nginx
```

---

## 🔹 Describe PVC

```bash
kubectl describe pvc my-pvc -n nginx
```

---

## 🔹 Check PV

```bash
kubectl get pv
```

---

## 🔹 Check StorageClass

```bash
kubectl get storageclass
```

---

# 🎯 Final Mental Model

```id="z5yltr"
PVC = Request (what app needs)
PV = Supply (what cluster has)
Binding = Match request with supply
Pod = Consumer
Kubelet = Mounts storage
Backend = Enforces real limits
```

---

# 🧠 Interview-Ready Summary

> PersistentVolumeClaim is a namespace-scoped resource that requests storage.
> It binds to a cluster-scoped PersistentVolume in a 1:1 relationship.
> Pods use PVCs to consume storage without needing to know underlying details.

---

If you want next, I can give:

* 🔥 StorageClass deep dive (VERY IMPORTANT)
* 🔥 CSI drivers internal working
* 🔥 StatefulSet + PVC real production architecture
