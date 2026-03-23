# 🟢 Kubernetes Service — Complete In-Depth Notes (Beginner → Advanced)

> A **Service** in Kubernetes provides **stable networking** to access Pods.

Pods are:

* Ephemeral (can die anytime)
* Get new IPs when recreated
* Not reliable to access directly

A **Service solves this problem**.

---

# 1️⃣ Why Service is Needed?

### ❌ Problem Without Service

You create Pods:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  containers:
    - name: app
      image: nginx
```

Pod gets IP:

```
10.244.1.5
```

If Pod dies → new Pod gets:

```
10.244.1.9
```

Your application breaks.

---

### ✅ Solution: Service

Service gives:

* Stable IP (ClusterIP)
* Stable DNS name
* Load balancing
* Pod discovery

---

# 2️⃣ What is a Service?

A Service is:

> An abstraction that exposes a set of Pods using a stable network endpoint.

It selects Pods using **labels**.

---

# 3️⃣ How Service Works Internally

Inside cluster:

1. Service created
2. Kubernetes assigns:

   * ClusterIP
   * DNS name
3. Service finds matching Pods using selector
4. kube-proxy sets up routing rules (iptables/ipvs)
5. Traffic is load balanced to Pods

---

# 4️⃣ Service Types

There are **4 main types**:

| Type         | Used For                     | External Access? |
| ------------ | ---------------------------- | ---------------- |
| ClusterIP    | Internal communication       | ❌                |
| NodePort     | External access via Node IP  | ✅                |
| LoadBalancer | Cloud external load balancer | ✅                |
| ExternalName | Map to external DNS          | Indirect         |

---

# 5️⃣ 1. ClusterIP (Default)

### Used For:

Internal Pod-to-Pod communication.

---

## Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 80
```

---

### How It Works

* Service gets Cluster IP like:

  ```
  10.96.45.12
  ```
* Only accessible inside cluster.
* DNS name:

  ```
  my-service.default.svc.cluster.local
  ```

---

### Internal Flow

Pod A → Service → One of matching Pods

---

# 6️⃣ 2. NodePort

### Used For:

Access application from outside cluster.

---

## Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
```

---

### What Happens

* Opens port **30007** on every node.
* Access using:

```
<NodeIP>:30007
```

Example:

```
192.168.1.10:30007
```

---

### Flow

User → NodeIP:NodePort → Service → Pod

---

# 7️⃣ 3. LoadBalancer

### Used For:

Cloud environments (AWS, Azure, GCP).

When created:

* Cloud provider creates external load balancer
* Assigns public IP

---

## Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-lb-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 80
```

---

### What Happens

Kubernetes talks to cloud API (like:

* Amazon Web Services
* Microsoft Azure
* Google Cloud Platform)

Cloud creates external Load Balancer.

User → Public IP → LB → Node → Pod

---

# 8️⃣ 4. ExternalName

Maps service to external DNS.

---

## Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: google-service
spec:
  type: ExternalName
  externalName: google.com
```

Now inside cluster:

```
google-service.default.svc.cluster.local
```

Points to:

```
google.com
```

No proxying. Only DNS alias.

---

# 9️⃣ Important Service Fields (Very Important)

## selector

Matches Pods using labels.

```yaml
selector:
  app: my-app
```

If labels don't match → Service won't work.

---

## port vs targetPort

| Field      | Meaning            |
| ---------- | ------------------ |
| port       | Service port       |
| targetPort | Pod container port |

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

Service listens on 80
Pod container listens on 8080

---

## nodePort

Used only in NodePort type.

Range:

```
30000–32767
```

---

# 🔟 DNS Inside Cluster

Every service gets DNS:

```
<service-name>.<namespace>.svc.cluster.local
```

Example:

```
my-service.default.svc.cluster.local
```

From same namespace:

```
http://my-service
```

---

# 1️⃣1️⃣ Headless Service

Used when:

* You don’t want load balancing
* You want direct Pod DNS
* Used in StatefulSets

---

## Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-headless
spec:
  clusterIP: None
  selector:
    app: my-app
  ports:
    - port: 80
```

Now DNS returns:

```
Pod1 IP
Pod2 IP
Pod3 IP
```

Instead of single Service IP.

---

# 1️⃣2️⃣ kube-proxy Role

Runs on every node.

It:

* Watches Service & Endpoints
* Creates iptables/ipvs rules
* Routes traffic to Pods

Without kube-proxy → Service won't work.

---

# 1️⃣3️⃣ Endpoints Object

When Service created, Kubernetes automatically creates:

```
Endpoints
```

Example:

```bash
kubectl get endpoints my-service
```

Shows Pod IPs behind Service.

Service → Endpoints → Pods

---

# 1️⃣4️⃣ Real Production Architecture Example

Frontend Pod
↓
ClusterIP Service
↓
Backend Pods
↓
Database Service
↓
Database Pod

External traffic:
User → LoadBalancer → Frontend Service

---

# 1️⃣5️⃣ Common Mistakes

❌ Labels mismatch
❌ Wrong targetPort
❌ Using NodePort in production
❌ Forgetting namespace
❌ Accessing ClusterIP from outside cluster

---

# 1️⃣6️⃣ When to Use What?

| Situation              | Use          |
| ---------------------- | ------------ |
| Internal microservices | ClusterIP    |
| Local testing          | NodePort     |
| Cloud production       | LoadBalancer |
| Stateful apps          | Headless     |
| External DB            | ExternalName |

---

# 1️⃣7️⃣ Service vs Ingress

Service:

* Layer 4 (TCP/UDP)

Ingress:

* Layer 7 (HTTP/HTTPS routing)
* Path-based routing
* Domain-based routing

---

# 1️⃣8️⃣ Complete Example (Deployment + Service)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
          image: nginx
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

Apply:

```bash
kubectl apply -f file.yaml
```

---

# 🧠 Final Understanding (Very Clear)

Pod = Temporary
Service = Stable Network Identity

Service:

* Gives stable IP
* Gives stable DNS
* Load balances
* Enables service discovery

---

If you want next:

* 🔥 Ingress full deep notes
* 🔥 kube-proxy internal working (iptables vs ipvs)
* 🔥 Real networking flow inside cluster
* 🔥 Service in production microservices architecture (like your CricHub system)
