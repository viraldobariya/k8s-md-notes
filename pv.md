# 🟢 PersistentVolume (PV) — Complete Long-Term Notes (Deep & Clear)

This is a **cluster-level storage concept** in Kubernetes.
We’ll cover everything clearly — architecture, lifecycle, types, scheduling, binding, and real-world behavior.

---

# 1️⃣ What is a PersistentVolume (PV)?

A **PersistentVolume (PV)** is:

> A cluster-scoped storage resource that represents real storage in the infrastructure.

It abstracts:

* Local disk
* Network storage (NFS)
* Cloud disks (EBS, GCE PD, Azure Disk)
* Distributed storage (Ceph, etc.)

---

## 🔎 Scope

| Resource | Scope              |
| -------- | ------------------ |
| PV       | 🌍 Cluster-level   |
| PVC      | 📦 Namespace-level |
| Pod      | 📦 Namespace-level |

PV exists independently of any namespace.

---

# 2️⃣ Why PV Exists

Pods are **ephemeral**:

* If Pod dies → its container filesystem is lost.

PV provides:

* Persistent data beyond Pod lifecycle.
* Decoupling between storage and workload.

---

# 3️⃣ Storage Architecture Flow (Cluster Level)

```
User creates PVC
        ↓
PersistentVolumeController watches PVC
        ↓
Finds matching PV (static)
OR
Triggers StorageClass (dynamic)
        ↓
PV ↔ PVC binding (1:1)
        ↓
Scheduler schedules Pod
        ↓
Kubelet mounts volume on Node
        ↓
Container sees mounted path
```

---

# 4️⃣ Static vs Dynamic Provisioning

## 🔹 Static Provisioning

Admin manually creates PV.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: static-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
```

Then developer creates PVC.

Kubernetes binds them if compatible.

---

## 🔹 Dynamic Provisioning (Modern Way)

Developer creates only PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-storage
spec:
  storageClassName: gp3
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

StorageClass provisions storage automatically.

---

# 5️⃣ PV Key Fields (Deep Explanation)

## 🔹 capacity

```yaml
capacity:
  storage: 10Gi
```

* Logical limit used for matching.
* Enforced only if backend supports quota (e.g., EBS).

For `hostPath`, not strictly enforced.

---

## 🔹 accessModes

Defines how volume can be mounted.

| Mode                | Meaning               | Multi-node? |
| ------------------- | --------------------- | ----------- |
| ReadWriteOnce (RWO) | One node read/write   | ❌           |
| ReadOnlyMany (ROX)  | Many nodes read-only  | ✅           |
| ReadWriteMany (RWX) | Many nodes read/write | ✅           |
| ReadWriteOncePod    | One Pod only          | ❌           |

⚠ Important: Access mode depends on backend capability.

Example:

* EBS → RWO
* NFS → RWX
* CephFS → RWX

---

## 🔹 volumeMode

```yaml
volumeMode: Filesystem
```

Options:

* Filesystem (default)
* Block (raw block device)

Block mode is used for databases needing raw disk.

---

## 🔹 persistentVolumeReclaimPolicy

What happens after PVC deletion?

| Policy  | Behavior          |
| ------- | ----------------- |
| Retain  | Keeps disk & data |
| Delete  | Deletes disk      |
| Recycle | Deprecated        |

Production databases usually use `Retain`.

---

## 🔹 storageClassName

Links PV to StorageClass.

If empty:

```yaml
storageClassName: ""
```

→ Static binding only.

---

# 6️⃣ PV Types (All Major Backends)

---

## 🔹 1. hostPath

```yaml
hostPath:
  path: /mnt/data
```

* Node-local directory
* No scheduler awareness
* Dev/testing only
* Not production safe

---

## 🔹 2. local (Proper Local Volume)

```yaml
local:
  path: /mnt/disks/ssd1
nodeAffinity:
  required:
    nodeSelectorTerms:
      - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
              - worker-1
```

* Node-bound storage
* Scheduler aware
* Safer than hostPath
* Still node-local

---

## 🔹 3. NFS

```yaml
nfs:
  server: 10.0.0.10
  path: /exports
```

* Shared storage
* Supports RWX
* Multi-node friendly

---

## 🔹 4. AWS EBS

Provisioned via CSI driver.

* Block storage
* RWO
* Cloud-managed disk

---

## 🔹 5. Azure Disk / GCE PD

Similar to EBS.
Cloud-managed block storage.

---

## 🔹 6. Ceph / Distributed Storage

Enterprise distributed storage.
Supports RWX.

---

# 7️⃣ PV Lifecycle States

| State     | Meaning                    |
| --------- | -------------------------- |
| Available | Not bound                  |
| Bound     | Attached to PVC            |
| Released  | PVC deleted, data retained |
| Failed    | Error state                |

Example:

```
Available → Bound → Released → (Admin action) → Available
```

---

# 8️⃣ Binding Logic (How Kubernetes Matches PV & PVC)

PVC must match:

* storageClassName
* accessModes
* requested storage ≤ PV capacity
* volumeMode

Kubernetes chooses smallest suitable PV.

Binding is:

```
1 PV ↔ 1 PVC
```

Never 1:many.

---

# 9️⃣ How Mounting Works Internally

After binding:

1. Scheduler selects node.
2. Scheduler checks:

   * Volume node affinity
   * Access mode compatibility
3. Kubelet on selected node:

   * Mounts disk to node filesystem
   * Attaches block device (cloud)
4. Container runtime mounts inside container.

So mounting happens at:

> Node level by Kubelet.

---

# 🔟 Multi-Node Behavior

## RWO Volume

* Only one node can mount read-write.
* Multiple Pods allowed only if on same node.

## RWX Volume

* Many nodes can mount simultaneously.

---

# 1️⃣1️⃣ What Happens When Pod Dies?

Nothing happens to PV.

Storage remains.

Pod restart → same PVC → same data.

---

# 1️⃣2️⃣ What Happens When PVC Is Deleted?

Depends on reclaim policy.

* Retain → PV stays Released
* Delete → Disk deleted
* Admin must clean if Retain

---

# 1️⃣3️⃣ Production Architecture

Cloud cluster (EKS example):

```
PVC created
   ↓
StorageClass (gp3)
   ↓
EBS CSI driver
   ↓
AWS API
   ↓
EBS volume created
   ↓
PV auto-created
   ↓
Bound to PVC
```

---

# 1️⃣4️⃣ Common Mistakes

❌ Thinking PV is namespace-level
❌ Thinking capacity always enforces quota
❌ Thinking RWO means single Pod
❌ Using hostPath in production

---

# 1️⃣5️⃣ Interview-Ready Summary

> PersistentVolume is a cluster-scoped abstraction of real storage.
> It binds 1:1 with a namespace-scoped PVC.
> Mounting occurs at node level via kubelet.
> Enforcement of size and access depends on backend storage system.

---

# 🧠 Final Mental Model

```
PV = Physical Storage Representation
PVC = Request for Storage
Pod = Consumer of Storage
Scheduler = Ensures correct node
Kubelet = Performs mount
Storage Backend = Enforces limits
```

---

If you want next, I can give equally deep notes on:

* PVC (deep dive)
* StorageClass & CSI drivers
* Volume expansion
* StatefulSet + storage architecture 🚀
