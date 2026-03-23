# 📦 Custom Resource Definitions (CRDs) in Kubernetes

> Deep • Clear • Production-Level • With Real Examples • No Ambiguity

---

# 1️⃣ Why CRDs Exist

Kubernetes supports built-in resources:

* Pods
* Deployments
* Services
* ConfigMaps

But real systems need custom domain objects like:

* Database
* KafkaCluster
* PaymentGateway
* CacheCluster
* AppEnvironment

Instead of managing everything using plain Deployments + Services, we extend Kubernetes.

That extension mechanism is:

> **Custom Resource Definition (CRD)**

---

# 2️⃣ What is a CRD?

A **CRD** allows you to:

> Create your own Kubernetes resource type.

After creating a CRD, Kubernetes behaves as if that resource is native.

Example:

After CRD creation, you can do:

```bash
kubectl get databases
kubectl apply -f mydb.yaml
```

Even though `Database` did not originally exist.

---

# 3️⃣ High-Level Architecture

CRD alone = only storage in etcd.

To make it intelligent, you need:

> CRD + Controller (Operator)

Flow:

Custom Resource (CR)
↓
Stored in etcd
↓
Controller watches
↓
Controller creates/updates real resources (Pods, Services, etc.)

---

# 4️⃣ Core Terminology (Very Important)

| Term       | Meaning                            |
| ---------- | ---------------------------------- |
| CRD        | Defines a new resource type        |
| CR         | An instance of that resource       |
| Controller | Logic that reacts to CR changes    |
| Operator   | CRD + Controller packaged together |

---

# 5️⃣ Real Example: Database CRD

Let’s design a real production example.

We want:

```yaml
apiVersion: data.mycompany.com/v1
kind: Database
metadata:
  name: orders-db
spec:
  engine: postgres
  version: "15"
  storage: 10Gi
```

This does NOT exist in Kubernetes by default.

So we define it.

---

# 6️⃣ Step 1 — Create the CRD

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.data.mycompany.com
spec:
  group: data.mycompany.com
  names:
    kind: Database
    plural: databases
    singular: database
    shortNames:
      - db
  scope: Namespaced
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                engine:
                  type: string
                version:
                  type: string
                storage:
                  type: string
```

---

# 7️⃣ Breaking It Down (No Confusion)

## 🔹 metadata.name

Must follow format:

```
<plural>.<group>
```

Here:

```
databases.data.mycompany.com
```

---

## 🔹 group

Acts like API namespace.

Example:

```
data.mycompany.com
```

Final API becomes:

```
data.mycompany.com/v1
```

---

## 🔹 names

Defines how kubectl interacts:

* kind → Database
* plural → databases
* singular → database
* shortNames → db

After applying CRD:

```bash
kubectl get databases
kubectl get db
```

Both work.

---

## 🔹 scope

Two options:

| Scope      | Meaning                      |
| ---------- | ---------------------------- |
| Namespaced | Resource exists in namespace |
| Cluster    | Resource exists cluster-wide |

Example:

Database → Namespaced
NodePool → Cluster

---

## 🔹 versions

CRDs support versioning.

You can define:

* v1
* v2
* v1beta1

Important fields:

| Field   | Meaning                |
| ------- | ---------------------- |
| served  | API available          |
| storage | Stored version in etcd |

---

## 🔹 schema (Very Important)

This defines validation.

If schema says:

```yaml
engine:
  type: string
```

Then:

```yaml
engine: 123
```

Will be rejected.

This is how you enforce structure.

---

# 8️⃣ Step 2 — Create a Custom Resource (CR)

After CRD is applied:

```yaml
apiVersion: data.mycompany.com/v1
kind: Database
metadata:
  name: orders-db
  namespace: production
spec:
  engine: postgres
  version: "15"
  storage: 10Gi
```

Apply:

```bash
kubectl apply -f orders-db.yaml
```

Now:

```bash
kubectl get databases -n production
```

Works.

But nothing happens yet.

Because no controller exists.

---

# 9️⃣ Why CRD Alone Is Not Enough

CRD only stores object in etcd.

It does NOT:

* Create pods
* Create statefulsets
* Provision storage

That logic comes from:

> Controller (Operator)

---

# 🔟 What Controller Does

For our Database example:

Controller logic:

IF Database created
→ Create StatefulSet
→ Create Service
→ Create PVC
→ Update status

That’s how operators like:

* Prometheus Operator
* ArgoCD
* Cert Manager

work internally.

---

# 1️⃣1️⃣ Status Subresource (Advanced)

You can separate:

* spec → desired state
* status → actual state

Example:

```yaml
status:
  phase: Running
  endpoint: orders-db.production.svc.cluster.local
```

Users modify spec.

Controller updates status.

Important production design principle.

---

# 1️⃣2️⃣ Cluster-Scoped CRD Example

Example: NodePool

```yaml
scope: Cluster
```

Then:

```bash
kubectl get nodepools
```

No namespace needed.

Used in:

* Autoscaler
* Cluster infrastructure management

---

# 1️⃣3️⃣ Versioning in CRDs (Advanced Production Topic)

Example:

```yaml
versions:
- name: v1alpha1
- name: v1beta1
- name: v1
```

Common lifecycle:

v1alpha1 → Experimental
v1beta1 → Stable but evolving
v1 → Production ready

You can also define conversion webhooks for version upgrades.

---

# 1️⃣4️⃣ Real Production Use Cases

| Product             | Uses CRD          |
| ------------------- | ----------------- |
| ArgoCD              | Application       |
| Cert Manager        | Certificate       |
| Prometheus Operator | ServiceMonitor    |
| Istio               | VirtualService    |
| Crossplane          | CompositeResource |

These are not built-in resources.

They are CRDs.

---

# 1️⃣5️⃣ When to Use CRDs

Use CRDs when:

✅ You want Kubernetes-style API
✅ You want declarative infrastructure
✅ You need custom automation logic
✅ You are building a platform

Do NOT use CRDs when:

❌ Simple configmap is enough
❌ No controller logic needed

---

# 1️⃣6️⃣ Comparison: CRD vs Built-in Resource

| Feature                    | Built-in | CRD          |
| -------------------------- | -------- | ------------ |
| Defined by Kubernetes      | ✅        | ❌            |
| Extendable by you          | ❌        | ✅            |
| Stored in etcd             | ✅        | ✅            |
| Needs controller for logic | Built-in | You build it |

---

# 1️⃣7️⃣ Internal Flow (Advanced Understanding)

When CR is created:

1. API Server validates against schema
2. Object stored in etcd
3. Controller watch triggers
4. Controller reconciles desired vs actual
5. Updates status

This is called:

> Reconciliation Loop

Core of Kubernetes design.

---

# 1️⃣8️⃣ Final Mental Model

CRD → Defines new API
CR → Instance of that API
Controller → Brain
Operator → CRD + Controller

---

# 1️⃣9️⃣ Real Interview-Level Understanding

Q: Does CRD automatically create Pods?
→ ❌ No

Q: Where are CRDs stored?
→ etcd

Q: Can CRD be cluster-scoped?
→ ✅ Yes

Q: What ensures CR validation?
→ openAPIV3Schema

---

If you want next:

* 🔥 How to build a simple controller
* 🔥 Operator pattern deep dive
* 🔥 CRD conversion webhooks
* 🔥 Admission webhooks vs CRDs
* 🔥 Real multi-tenant CRD architecture

Tell me what level you want next 🚀
