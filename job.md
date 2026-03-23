# 🧾 Kubernetes Job — Complete Notes

---

## 📌 What is a Job?

A **Job** ensures:

> 🎯 A task runs to completion successfully.

Unlike Deployment (runs forever),
Job runs Pods **until they finish successfully**.

---

## 🧠 When to Use Job

* Batch processing
* DB migration
* Backup scripts
* Data processing
* One-time tasks

---

## 📄 Basic Job YAML

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: simple-job
spec:
  completions: 1
  parallelism: 1
  template:
    spec:
      containers:
        - name: busybox
          image: busybox
          command: ["echo", "Hello Kubernetes"]
      restartPolicy: Never
```

---

## 🔹 Key Fields Explained

### `completions`

Total successful executions required.

Example:

```
completions: 5
```

→ Job must succeed 5 times.

---

### `parallelism`

How many Pods run at the same time.

Example:

```
parallelism: 2
completions: 6
```

Flow:

* 2 run in parallel
* When finished, next 2 start
* Continues until 6 successful runs

---

### `restartPolicy`

Allowed values:

```
Never
OnFailure
```

⚠️ `Always` not allowed in Jobs.

---

## 🔄 Job Execution Logic

```text
If Pod fails → retry (depending on backoffLimit)
If Pod succeeds → count towards completions
When completions met → Job finishes
```

---

## 🔥 Important Fields

### `backoffLimit`

Number of retries before marking Job as failed.

```yaml
backoffLimit: 3
```

---

### `activeDeadlineSeconds`

Time limit for Job execution.

```yaml
activeDeadlineSeconds: 100
```

---

## 📊 Check Job Status

```bash
kubectl get jobs
kubectl describe job simple-job
kubectl get pods
```

---

## 🧨 What Happens After Completion?

By default:

* Completed Pods remain.

You can clean automatically:

```yaml
ttlSecondsAfterFinished: 30
```

---

## 🆚 Job vs Deployment

| Feature                 | Job         | Deployment        |
| ----------------------- | ----------- | ----------------- |
| Runs forever            | ❌           | ✅                 |
| Runs to completion      | ✅           | ❌                 |
| Auto restart on failure | Limited     | Yes               |
| Rolling updates         | ❌           | ✅                 |
| Use case                | Batch tasks | Long-running apps |

---

## 🆚 Job vs CronJob

| Job            | CronJob            |
| -------------- | ------------------ |
| Runs once      | Runs on schedule   |
| Manual trigger | Time-based trigger |

---

## 🎯 Types of Jobs

### 1️⃣ Single Completion Job

```
completions: 1
parallelism: 1
```

### 2️⃣ Fixed Completion Count Job

```
completions: 5
parallelism: 2
```

### 3️⃣ Work Queue Job

```
completions not set
parallelism: N
```

Used with distributed workers.

---

## 🧠 Mental Model

```text
Deployment → Keep app running
ReplicaSet → Keep N pods running
Job → Run until success
```

---

## 🎯 Interview One-Liner

> A Job ensures one or more Pods run to successful completion, typically used for batch or one-time tasks.

---

If you want next:

* 🔥 CronJob notes
* 🔥 Parallel Job deep explanation
* 🔥 Real production batch architecture usage
