# 🟢 Kubernetes Storage — PersistentVolume, PVC & Mounting (Deep, Clear, Complete Notes)

This is a **clean architectural understanding** of Kubernetes storage.

We’ll cover:

* What PV really is (internals)
* What PVC really does
* How binding works
* How scheduler + kubelet interact
* All major volume types (hostPath, local, NFS, cloud, CSI)
* Real production behavior
* Common confusion clarified

No ambiguity. Clear mental model.

---

# 1️⃣ Core Idea — Why Kubernetes Needs PV

Containers are **ephemeral**.

When a Pod dies:

* Container filesystem is gone
* Data inside `/var/lib/...` disappears

Kubernetes separates:

```
Compute  ≠  Storage
```

So storage survives Pod lifecycle.

That separation is implemented using:

```
PV  → Actual storage resource
PVC → Request for storage
Pod → Consumer
```

---

# 2️⃣ What is a PersistentVolume (PV)?

A **PersistentVolume (PV)** is:

> A cluster-scoped API object that represents real storage in infrastructure.

It does NOT store data itself.

It represents:

* A disk
* A directory
* A network filesystem
* A cloud block volume
* A distributed storage system

---

## 🔎 Scope Clarification

| Object | Scope           |
| ------ | --------------- |
| PV     | 🌍 Cluster-wide |
| PVC    | 📦 Namespace    |
| Pod    | 📦 Namespace    |

PV is not owned by any namespace.

---

# 3️⃣ What is a PersistentVolumeClaim (PVC)?

A PVC is:

> A namespace-scoped request for storage.

It says:

* I need 5Gi
* I need RWO
* I need this StorageClass

PVC does NOT know:

* Which disk
* Which node
* Which cloud volume

That decision happens automatically.

---

# 4️⃣ Complete Storage Flow (End-to-End)

Here is the real flow inside Kubernetes:

```
Developer creates PVC
        ↓
PV Controller watches PVC
        ↓
If static → Find matching PV
If dynamic → Trigger StorageClass provisioner
        ↓
PV is bound to PVC (1:1)
        ↓
Pod references PVC
        ↓
Scheduler evaluates:
    - NodeAffinity (if any)
    - Access mode
    - Volume constraints
        ↓
Pod scheduled to correct node
        ↓
Kubelet on node:
    - Attaches disk (if cloud)
    - Mounts volume to node
    - Mounts inside container
```

Important:

> Scheduler does NOT mount.
> Kubelet mounts.

---

# 5️⃣ Static vs Dynamic Provisioning

---

## 🔹 Static Provisioning

Admin creates PV manually.

Example:

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

PVC must match it.

Best for:

* On-prem clusters
* Local disks
* Controlled environments

---

## 🔹 Dynamic Provisioning (Modern Cloud Way)

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

StorageClass triggers provisioner (CSI driver).

Cloud example flow:

```
PVC → StorageClass → CSI Driver → Cloud API → Disk Created → PV auto-created → Bound
```

---

# 6️⃣ Deep Dive — PV Spec Fields Explained

---

## 🔹 capacity

Logical size for matching.

```
capacity:
  storage: 10Gi
```

Important:

* Enforced only if backend supports it.
* hostPath does NOT enforce.
* Cloud disks DO enforce.

---

## 🔹 accessModes

Defines mounting capability.

| Mode             | Meaning               | Multi-node? |
| ---------------- | --------------------- | ----------- |
| RWO              | One node read-write   | ❌           |
| ROX              | Many nodes read-only  | ✅           |
| RWX              | Many nodes read-write | ✅           |
| ReadWriteOncePod | One Pod only          | ❌           |

Important:

> RWO = One NODE, not one Pod.

---

## 🔹 volumeMode

```
volumeMode: Filesystem
```

Options:

* Filesystem (default)
* Block (raw device)

Block mode is used by databases needing direct block device.

---

## 🔹 persistentVolumeReclaimPolicy

After PVC deletion:

| Policy  | Behavior    |
| ------- | ----------- |
| Retain  | Keep disk   |
| Delete  | Delete disk |
| Recycle | Deprecated  |

Production databases usually use Retain.

---

# 7️⃣ Volume Types — Deep Comparison

---

# 🔴 1. hostPath

```yaml
hostPath:
  path: /mnt/data
```

### Behavior:

* Mounts node directory directly.
* No storage abstraction.
* Scheduler unaware (unless manually restricted).

### Problems:

* Security risk
* Not portable
* No true orchestration

### Use Only For:

* Dev
* Minikube
* Kind
* Testing

Never recommended for production multi-node clusters.

---

# 🟡 2. local (Proper Local Volume)

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

### Key Characteristics:

* Node-local storage
* Must define nodeAffinity
* Scheduler aware
* Production-grade for bare-metal

### What Happens:

PVC binds → Scheduler sees nodeAffinity → Pod scheduled to correct node automatically.

### Use Case:

* Bare-metal clusters
* High-performance SSD databases
* Stateful workloads

Still risky if node dies (data lost unless disk recovered).

---

# 🟢 3. NFS (Network File System)

```yaml
nfs:
  server: 10.0.0.10
  path: /exports
```

### Characteristics:

* Shared storage
* Supports RWX
* Multi-node friendly
* No node affinity required

### Use Case:

* Shared logs
* Shared app uploads
* Multi-replica apps

---

# 🔵 4. Cloud Block Storage (AWS EBS, Azure Disk, GCE PD)

Provisioned via CSI drivers.

Example concept:

* PVC requests 10Gi
* CSI calls cloud API
* Disk created
* Attached to node
* Mounted

Characteristics:

* RWO
* Cloud-managed
* Highly durable
* Production standard

Node scheduling respects attachment limits.

---

# 🟣 5. Distributed Storage (Ceph, Portworx, etc.)

Cluster-wide storage system.

Supports:

* RWX
* Multi-node access
* High availability

Best for enterprise clusters.

---

# 8️⃣ Binding Logic — How PV Matches PVC

Matching requires:

* Same storageClassName
* Compatible accessModes
* PVC request ≤ PV capacity
* Same volumeMode

Binding is always:

```
1 PV ↔ 1 PVC
```

Never shared.

---

# 9️⃣ Scheduler + Storage Awareness

Scheduler checks:

* PV nodeAffinity (local)
* Volume topology constraints (cloud)
* Access mode conflicts
* Volume attachment limits

Important:

Scheduler decides node.
Kubelet performs mount.

---

# 🔟 Mounting Internals

On selected node:

1. CSI driver attaches disk (if cloud).
2. Kubelet mounts to:

   ```
   /var/lib/kubelet/pods/<pod-id>/volumes/
   ```
3. Container runtime bind-mounts inside container.

Mounting happens at OS level.

---

# 1️⃣1️⃣ What Happens When Node Dies?

Depends on backend:

| Backend     | Behavior                  |
| ----------- | ------------------------- |
| hostPath    | Data lost                 |
| local       | Disk stuck on dead node   |
| NFS         | Pod can restart elsewhere |
| EBS         | Reattach to new node      |
| Distributed | Survives                  |

---

# 1️⃣2️⃣ StatefulSet + Storage

StatefulSet creates:

```
pod-0 → pvc-0 → pv-0
pod-1 → pvc-1 → pv-1
```

Each replica gets its own volume.

Common with:

* Databases
* Kafka
* Elasticsearch

---

# 1️⃣3️⃣ Common Misconceptions

❌ PV belongs to namespace
❌ RWO means single Pod
❌ hostPath is same as local
❌ capacity always enforces quota
❌ Scheduler mounts volumes

---

# 🧠 Final Mental Model (Very Important)

```
PV  = Infrastructure Storage Object
PVC = Contract / Request
Pod = Consumer
Scheduler = Decides correct node
Kubelet = Attaches + Mounts
CSI Driver = Talks to storage backend
Backend = Actually stores data
```

---

# 🎯 Architecture Comparison Summary

| Type        | Scheduler Aware | Multi-node | Production Safe |
| ----------- | --------------- | ---------- | --------------- |
| hostPath    | ❌               | ❌          | ❌               |
| local       | ✅               | ❌          | ⚠ Bare-metal    |
| NFS         | ✅               | ✅          | ✅               |
| Cloud (EBS) | ✅               | ❌ (RWO)    | ✅               |
| Distributed | ✅               | ✅          | ✅               |

---

# 🏁 Interview-Ready Definition

> PersistentVolume is a cluster-scoped abstraction of real infrastructure storage.
> It binds one-to-one with a PVC.
> Scheduler ensures correct node placement.
> Kubelet performs mount operations.
> Actual durability and behavior depend entirely on backend storage type.

---

If you want next level deep dive, we can go into:

* CSI architecture (internals)
* Volume expansion flow
* Topology-aware scheduling
* How attachment limits affect scheduling
* Real production storage design patterns 🚀
