# 🐳 `kubectl create` vs `kubectl apply` — Clear & In-Depth Notes

Both commands create resources in Kubernetes, but they behave differently.

---

# 🔹 1️⃣ `kubectl create`

### ✅ Purpose

Creates a **new resource** in the cluster.

### 🔹 Behavior

* Fails if the resource **already exists**
* Imperative style (quick commands)
* Does **NOT** track configuration changes automatically

### 🔹 Example

```bash
kubectl create -f deployment.yaml
```

Or imperative:

```bash
kubectl create deployment nginx --image=nginx
```

### 🔹 When to Use

* Quick testing
* One-time resource creation
* Imperative commands (exam scenarios)

---

# 🔹 2️⃣ `kubectl apply`

### ✅ Purpose

Creates **or updates** a resource.

### 🔹 Behavior

* If resource doesn’t exist → **creates**
* If resource exists → **updates (patches)** it
* Declarative style
* Tracks last applied configuration (via annotation)

### 🔹 Example

```bash
kubectl apply -f deployment.yaml
```

### 🔹 When to Use

* Production
* CI/CD pipelines
* Infrastructure as Code
* Re-applying updated YAML files

---

# 🔥 Key Technical Difference

| Feature                | `create`  | `apply` |
| ---------------------- | --------- | ------- |
| Creates resource       | ✅         | ✅       |
| Updates existing       | ❌ (fails) | ✅       |
| Declarative management | ❌         | ✅       |
| Safe for repeated runs | ❌         | ✅       |
| Best for production    | ❌         | ✅       |

---

# 🧠 Internal Concept

* `create` → sends full object to API server once.
* `apply` → performs **3-way merge**:

  * Last applied config
  * Current cluster state
  * New YAML file

This makes `apply` safer and smarter for updates.

---

# 🎯 Interview / Exam One-Liner

> `create` is imperative and fails if resource exists.
> `apply` is declarative and can create or update resources safely.

---

If you want, I can also explain:

* `apply` vs `replace`
* `apply` vs `patch`
* What is 3-way merge internally
* Server-side apply vs client-side apply

Just tell me 🚀
