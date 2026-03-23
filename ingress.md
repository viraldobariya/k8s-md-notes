# 🟣 Kubernetes Ingress — Complete In-Depth Notes (Beginner → Advanced)

> **Ingress** manages **HTTP/HTTPS (Layer 7)** traffic into your cluster.

If **Service = stable internal networking**,
then **Ingress = smart external HTTP router**.

---

# 1️⃣ Why Ingress is Needed?

Without Ingress:

* You expose each app using:

  * `NodePort` ❌ (not production-ready)
  * `LoadBalancer` ❌ (one LB per service → expensive)

If you have 10 services → 10 LoadBalancers 😵

---

### ✅ Ingress Solves This

* Single external IP
* Multiple domain routing
* Path-based routing
* TLS (HTTPS)
* Central traffic control

---

# 2️⃣ Important Concept: Ingress ≠ Ingress Controller

This is where beginners get confused.

| Component          | What It Is                                 |
| ------------------ | ------------------------------------------ |
| Ingress            | YAML rule definition                       |
| Ingress Controller | Actual reverse proxy that implements rules |

Ingress by itself does **nothing**.

You must install an Ingress Controller.

Popular controllers:

* NGINX (Most common)
* Traefik
* HAProxy

---

# 3️⃣ High-Level Architecture

```
User → Internet
      ↓
External LoadBalancer / NodePort
      ↓
Ingress Controller (NGINX)
      ↓
Service
      ↓
Pods
```

---

# 4️⃣ Real Example Scenario (Clear & Practical)

Let’s say you have:

| App         | Service Name | Port |
| ----------- | ------------ | ---- |
| Frontend    | frontend-svc | 80   |
| Backend API | api-svc      | 8080 |

You want:

```
example.com → frontend
example.com/api → backend
```

---

# 5️⃣ Step 1 — Deployment + Services

## Frontend Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 80
```

---

## Backend Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-svc
spec:
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
```

---

# 6️⃣ Step 2 — Create Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-svc
                port:
                  number: 80
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-svc
                port:
                  number: 80
```

---

# 7️⃣ What This Does

When user visits:

### 🔹 example.com/

→ Goes to frontend-svc
→ Goes to frontend Pods

### 🔹 example.com/api

→ Goes to api-svc
→ Goes to backend Pods

One domain. Multiple services. Clean routing.

---

# 8️⃣ Path Types (Very Important)

| Type                   | Meaning                 |
| ---------------------- | ----------------------- |
| Prefix                 | Matches based on prefix |
| Exact                  | Exact match only        |
| ImplementationSpecific | Controller decides      |

Example:

```yaml
path: /api
pathType: Prefix
```

Matches:

```
/api
/api/users
/api/v1/data
```

---

# 9️⃣ TLS (HTTPS Setup)

You can enable HTTPS easily.

First create secret:

```bash
kubectl create secret tls tls-secret \
  --cert=cert.crt \
  --key=cert.key
```

Then modify Ingress:

```yaml
spec:
  tls:
    - hosts:
        - example.com
      secretName: tls-secret
```

Now:

```
https://example.com
```

---

# 🔟 Multiple Domains Example

```yaml
rules:
  - host: app.com
    http:
      paths:
        - path: /
          backend:
            service:
              name: frontend-svc
              port:
                number: 80

  - host: api.app.com
    http:
      paths:
        - path: /
          backend:
            service:
              name: api-svc
              port:
                number: 80
```

Now:

```
app.com → frontend
api.app.com → backend
```

---

# 1️⃣1️⃣ How Ingress Controller Works Internally

If using NGINX controller:

1. Controller watches Ingress resources
2. Converts YAML rules → NGINX config
3. Reloads NGINX
4. Routes traffic accordingly

So internally it's just:

```
NGINX reverse proxy rules
```

---

# 1️⃣2️⃣ Ingress vs Service (Clear Difference)

| Feature         | Service  | Ingress   |
| --------------- | -------- | --------- |
| Layer           | L4 (TCP) | L7 (HTTP) |
| Load balancing  | Yes      | Yes       |
| Domain routing  | No       | Yes       |
| Path routing    | No       | Yes       |
| TLS termination | No       | Yes       |

---

# 1️⃣3️⃣ Production Architecture Example

For your microservices system (like CricHub idea):

```
User
 ↓
Cloud LoadBalancer
 ↓
Ingress Controller
 ↓
Auth Service
Match Service
Team Service
Payment Service
 ↓
Pods
```

Single public entry point. Clean routing.

---

# 1️⃣4️⃣ Common Mistakes

❌ Ingress created but controller not installed
❌ Wrong service name
❌ Wrong service port
❌ Forgetting DNS mapping
❌ Trying to use Ingress for TCP (not HTTP)

---

# 1️⃣5️⃣ Important Commands

Check ingress:

```bash
kubectl get ingress
```

Describe:

```bash
kubectl describe ingress app-ingress
```

Check controller pod:

```bash
kubectl get pods -n ingress-nginx
```

---

# 1️⃣6️⃣ When NOT to Use Ingress

Don’t use Ingress if:

* You need raw TCP/UDP routing
* You only have one simple service
* Internal-only traffic

---

# 🧠 Final Understanding (Very Clear)

Pod = temporary
Service = stable internal networking
Ingress = smart HTTP entry point

Ingress:

* Routes based on domain
* Routes based on path
* Handles HTTPS
* Centralizes external access

---

If you want next:

* 🔥 NGINX Ingress installation in minikube/kind
* 🔥 Ingress annotations explained
* 🔥 Real traffic flow deep networking explanation
* 🔥 API Gateway vs Ingress comparison
