# 🟢 ConfigMap & Secret in Kubernetes — Complete In-Depth Notes (Clear + Practical)

> Both **ConfigMap** and **Secret** are used to store configuration data outside your container image.

Instead of hardcoding config inside Docker image ❌
We externalize configuration ✅

---

# 1️⃣ Why Do We Need ConfigMap & Secret?

Imagine your app needs:

* DB host
* DB username
* Feature flags
* API keys
* JWT secret
* Passwords

If you bake these into the image:

* Not flexible ❌
* Not secure ❌
* Need rebuild on every change ❌

Kubernetes solves this with:

| Object    | Used For           |
| --------- | ------------------ |
| ConfigMap | Non-sensitive data |
| Secret    | Sensitive data     |

---

# 2️⃣ ConfigMap (Non-Sensitive Configuration)

> Stores plain-text configuration data.

Used for:

* Environment variables
* Config files
* App properties
* Feature flags

---

## 2.1 Example — Simple ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_HOST: mysql-service
  DB_PORT: "3306"
  APP_MODE: production
```

---

## 2.2 Use ConfigMap as Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-demo
spec:
  containers:
    - name: app
      image: nginx
      envFrom:
        - configMapRef:
            name: app-config
```

Inside container:

```bash
echo $DB_HOST
```

Output:

```
mysql-service
```

---

## 2.3 Use ConfigMap as File (Volume Mount)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-file-demo
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

Now files created:

```id="hmr8x3"
/etc/config/DB_HOST
/etc/config/DB_PORT
```

Each key becomes a file.

---

# 3️⃣ Secret (Sensitive Data)

> Stores confidential data like passwords, tokens, certificates.

Examples:

* DB password
* API key
* OAuth token
* TLS certificate

---

# 3.1 Important: Secrets Are Base64 Encoded

Not encrypted by default (unless etcd encryption enabled).

---

## 3.2 Create Secret via YAML

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQxMjM=
```

`cGFzc3dvcmQxMjM=` = base64 of `password123`

---

## 3.3 Create Secret via Command

```bash
kubectl create secret generic db-secret \
  --from-literal=DB_PASSWORD=password123
```

---

## 3.4 Use Secret as Environment Variable

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: DB_PASSWORD
```

---

## 3.5 Use Secret as Volume

```yaml
volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
```

Mount like ConfigMap.

---

# 4️⃣ ConfigMap vs Secret (Clear Comparison)

| Feature     | ConfigMap  | Secret          |
| ----------- | ---------- | --------------- |
| Data type   | Plain text | Sensitive       |
| Encoding    | Normal     | Base64          |
| Use case    | App config | Passwords, keys |
| Security    | Not secure | Slightly better |
| TLS support | No         | Yes             |

---

# 5️⃣ Real Example — Backend API with DB

Imagine:

* API connects to database like MySQL
* Needs host & password

---

## Step 1 — ConfigMap (Non-sensitive)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: backend-config
data:
  DB_HOST: mysql-service
  DB_PORT: "3306"
```

---

## Step 2 — Secret (Sensitive)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: backend-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQ=
```

---

## Step 3 — Use in Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: my-api:latest
          env:
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: backend-config
                  key: DB_HOST
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: backend-secret
                  key: DB_PASSWORD
```

---

# 6️⃣ Important Concepts

---

## 6.1 Updating ConfigMap

If mounted as:

* Environment variable → Pod restart required
* Volume → File updates automatically (within few seconds)

---

## 6.2 Secret Types

| Type                           | Used For             |
| ------------------------------ | -------------------- |
| Opaque                         | Generic secrets      |
| kubernetes.io/tls              | TLS certs            |
| kubernetes.io/dockerconfigjson | Docker registry auth |

---

## 6.3 TLS Secret Example

```bash
kubectl create secret tls tls-secret \
  --cert=cert.crt \
  --key=key.key
```

Used in Ingress for HTTPS.

---

# 7️⃣ Security Best Practices

❌ Don’t store secrets in Git
❌ Don’t hardcode passwords
❌ Don’t print secrets in logs

✅ Enable etcd encryption
✅ Use RBAC to restrict access
✅ Use external secret managers in production

Example external systems:

* HashiCorp (Vault)
* Amazon Web Services (Secrets Manager)

---

# 8️⃣ How Data Flows

```text
ConfigMap / Secret
        ↓
Kubelet
        ↓
Pod (Env or Volume)
        ↓
Application
```

---

# 9️⃣ Common Mistakes

❌ Using ConfigMap for passwords
❌ Forgetting base64 encoding
❌ Not restarting Pod after update
❌ Mixing up key names
❌ Assuming Secret is encrypted

---

# 🔟 When to Use What?

| Scenario      | Use       |
| ------------- | --------- |
| DB host       | ConfigMap |
| Feature flags | ConfigMap |
| Password      | Secret    |
| API token     | Secret    |
| TLS cert      | Secret    |

---

# 🧠 Final Understanding

ConfigMap:

> External configuration (non-sensitive)

Secret:

> Secure configuration (sensitive)

Both:

* Decouple config from image
* Make apps portable
* Allow environment-specific setup

---

# 🎯 One-Line Summary

> ConfigMap = configuration
> Secret = sensitive configuration

---

If you want next:

* 🔥 ConfigMap & Secret in real microservices architecture (like your CricHub backend)
* 🔥 External Secrets Operator explanation
* 🔥 Interview questions on ConfigMap & Secret
* 🔥 Deep security explanation (etcd encryption, RBAC)
