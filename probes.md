# 🟢 Kubernetes Probes — Complete In-Depth Notes (Clear + Practical)

> **Probes** are health checks used by Kubernetes to know the state of a container.

They help Kubernetes decide:

* Should traffic go to this Pod?
* Should this container be restarted?
* Is the app fully started?

---

# 1️⃣ Why Probes Are Needed?

Imagine your app:

* Takes 30 seconds to start
* Sometimes hangs
* Sometimes crashes silently

Without probes:

* Service still sends traffic ❌
* Users get errors ❌
* Pod may never restart ❌

With probes:

* Kubernetes detects problem
* Stops traffic
* Restarts container automatically

---

# 2️⃣ Types of Probes (Very Important)

There are **3 probes**:

| Probe           | Purpose             | What It Controls    |
| --------------- | ------------------- | ------------------- |
| Liveness Probe  | Is container alive? | Restart container   |
| Readiness Probe | Is container ready? | Service traffic     |
| Startup Probe   | Has app started?    | Delays other probes |

---

# 3️⃣ 1️⃣ Liveness Probe

> Detects if container is stuck or dead.

If liveness fails:

```
Container is restarted
```

---

## Example — HTTP Liveness

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-demo
spec:
  containers:
    - name: app
      image: nginx
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 5
```

---

### What Happens?

* Wait 10 seconds
* Every 5 seconds call `/`
* If HTTP not 200–399 → restart container

---

# 4️⃣ 2️⃣ Readiness Probe

> Determines if Pod should receive traffic.

If readiness fails:

```
Pod removed from Service endpoints
```

But container is NOT restarted.

---

## Example — Readiness Probe

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 3
```

---

### Real Use Case

Your app:

* Starts fast
* But DB connection not ready

Readiness probe:

* Fails until DB connected
* Traffic not sent
* Once ready → traffic starts

---

# 5️⃣ 3️⃣ Startup Probe

> Used for slow starting applications.

Without startup probe:

* Liveness might restart container before app starts

Startup probe disables liveness & readiness until it succeeds.

---

## Example

```yaml
startupProbe:
  httpGet:
    path: /health
    port: 80
  failureThreshold: 30
  periodSeconds: 10
```

Meaning:

* Try 30 times
* Every 10 seconds
* Gives 5 minutes startup time

---

# 6️⃣ Probe Methods

Kubernetes supports 3 types of checks:

| Type      | Used For                     |
| --------- | ---------------------------- |
| httpGet   | HTTP endpoint                |
| tcpSocket | TCP connection check         |
| exec      | Run command inside container |

---

# 7️⃣ Exec Probe Example

```yaml
livenessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

If file not exists → restart container.

---

# 8️⃣ TCP Probe Example

```yaml
livenessProbe:
  tcpSocket:
    port: 3306
  initialDelaySeconds: 10
  periodSeconds: 5
```

Useful for DB like:

* MySQL
* PostgreSQL

It checks if port is open.

---

# 9️⃣ Important Probe Parameters

| Field               | Meaning                   |
| ------------------- | ------------------------- |
| initialDelaySeconds | Wait before first check   |
| periodSeconds       | Check interval            |
| timeoutSeconds      | Probe timeout             |
| failureThreshold    | Fail after X failures     |
| successThreshold    | Success after X successes |

---

# 🔟 Real Production Example (API + DB)

Imagine:

* Backend API
* Connects to database
* Needs time to warm up

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: my-api:latest
          ports:
            - containerPort: 8080

          startupProbe:
            httpGet:
              path: /health
              port: 8080
            failureThreshold: 30
            periodSeconds: 5

          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5

          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
```

---

# 1️⃣1️⃣ Traffic Flow With Readiness

```text
User
 ↓
Service
 ↓
Only Ready Pods
 ↓
Container
```

If readiness fails:

* Pod removed from Service endpoints
* No traffic sent

---

# 1️⃣2️⃣ What Happens When Liveness Fails?

```text
Probe fails
 ↓
Kubelet detects failure
 ↓
Container restarted
```

Important:
Only container restarts, not whole Pod.

---

# 1️⃣3️⃣ Common Mistakes

❌ Not using readiness probe
❌ Using liveness instead of readiness
❌ Too small timeout
❌ Too aggressive restart configuration
❌ No startup probe for slow apps

---

# 1️⃣4️⃣ When to Use Which?

| Situation            | Use       |
| -------------------- | --------- |
| App hangs sometimes  | Liveness  |
| DB not ready yet     | Readiness |
| Slow boot (JVM apps) | Startup   |

---

# 1️⃣5️⃣ Real Systems Using Probes

Modern systems like:

* Spring Boot (`/actuator/health`)
* Express.js (custom `/health` endpoint)
* Django (health view)

Typically expose:

```
/health
/ready
```

---

# 🧠 Final Understanding (Very Clear)

Liveness:

> Should this container be restarted?

Readiness:

> Should traffic go to this Pod?

Startup:

> Has the application finished booting?

---

# 🎯 One-Line Summary

> Probes allow Kubernetes to automatically manage container health and traffic routing.

---

If you want next:

* 🔥 Probes interview questions
* 🔥 Probes + HPA interaction
* 🔥 Real debugging of failing probes
* 🔥 Production best practices settings
