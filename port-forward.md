## 🔹 `kubectl port-forward` — Beginner Clear Notes

### 📌 What It Does

`kubectl port-forward` creates a **temporary tunnel** from:

> 💻 Your Local Machine ➜ 📦 Pod / Service inside Kubernetes Cluster

It lets you access internal apps **without exposing them using a Service (NodePort/LoadBalancer).**

---

## 🧠 Why We Use It

Used mainly for:

* Debugging applications
* Accessing internal DB (MySQL, MongoDB)
* Testing APIs
* Accessing dashboards

---

## 🛠 Basic Syntax

```bash
kubectl port-forward <resource-type>/<name> <local-port>:<container-port>
```

OR

```bash
kubectl port-forward pod/<pod-name> 8080:80
```

---

## 📦 Example 1 — Port Forward to Pod

```bash
kubectl port-forward pod/nginx-pod 8080:80
```

👉 What happens:

* Local machine port **8080**
* Forwards to container port **80**
* Now open:

```
http://localhost:8080
```

You can access Nginx.

---

## 📦 Example 2 — Port Forward to Service (Recommended)

```bash
kubectl port-forward service/nginx-service 8080:80
```

This is better because:

* Service is stable
* Pod name may change
* Works even if pods restart

---

## 📦 Example 3 — Forward Database

```bash
kubectl port-forward pod/mysql-pod 3306:3306
```

Now connect from local:

```bash
mysql -h 127.0.0.1 -P 3306 -u root -p
```

---

## 🔹 Namespace Example

If resource is in another namespace:

```bash
kubectl port-forward pod/nginx-pod 8080:80 -n dev
```

---

## 🔹 Run in Background

By default, it runs in foreground.

Stop with:

```
Ctrl + C
```

To run in background:

```bash
kubectl port-forward pod/nginx-pod 8080:80 &
```

---

## 🔥 Important Things to Remember

* ✅ Only works while command is running
* ✅ Only accessible from your machine
* ❌ Not for production exposure
* ❌ Not load-balanced
* ❌ Not persistent

---

## 🎯 When NOT to Use It

For production traffic, use:

* ClusterIP (internal)
* NodePort
* LoadBalancer
* Ingress

---

## 🚀 Simple Mental Model

Think of `kubectl port-forward` as:

> 🔌 Temporary USB cable connecting your laptop to a pod.

---

If you want, I can now explain:

* 🔥 How port-forward works internally (API Server proxy)
* 🔥 Difference between port-forward vs NodePort
* 🔥 Interview questions on port-forward
