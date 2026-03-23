# ⏰ Kubernetes CronJob — Complete Notes

---

## 📌 What is a CronJob?

A **CronJob** creates **Jobs on a schedule**, similar to Linux `cron`.

> 🎯 Run tasks automatically at fixed time intervals.

---

## 🧠 When to Use

* Nightly database backup
* Daily report generation
* Cleanup tasks
* Periodic health checks
* Scheduled data sync

---

## 🏗 Architecture

```text
CronJob
   ↓ creates (on schedule)
Job
   ↓ creates
Pods
```

CronJob does NOT run Pods directly.
It creates a **Job**, and the Job runs Pods.

---

## 📄 Basic CronJob YAML

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: simple-cronjob
spec:
  schedule: "*/1 * * * *"   # every 1 minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: busybox
              image: busybox
              command: ["echo", "Hello from CronJob"]
          restartPolicy: OnFailure
```

---

## 🔹 Cron Schedule Format

```
* * * * *
| | | | |
| | | | └── Day of week (0–7)
| | | └──── Month (1–12)
| | └────── Day of month (1–31)
| └──────── Hour (0–23)
└────────── Minute (0–59)
```

### Examples

| Schedule      | Meaning               |
| ------------- | --------------------- |
| `"* * * * *"` | Every minute          |
| `"0 * * * *"` | Every hour            |
| `"0 0 * * *"` | Every day at midnight |
| `"0 0 * * 0"` | Every Sunday          |

---

## 🔥 Important Fields

### `schedule`

Defines when Job runs.

---

### `jobTemplate`

Defines the Job that gets created.

---

### `concurrencyPolicy`

Controls overlapping runs:

```yaml
concurrencyPolicy: Allow
```

Options:

| Value   | Meaning                           |
| ------- | --------------------------------- |
| Allow   | Default, allow multiple runs      |
| Forbid  | Skip if previous run still active |
| Replace | Cancel old job and start new      |

---

### `successfulJobsHistoryLimit`

How many successful Jobs to keep:

```yaml
successfulJobsHistoryLimit: 3
```

---

### `failedJobsHistoryLimit`

How many failed Jobs to keep:

```yaml
failedJobsHistoryLimit: 1
```

---

### `startingDeadlineSeconds`

If Job misses schedule (e.g., cluster down),
how long to wait before skipping it.

---

## 🔄 How It Works Internally

At scheduled time:

1. CronJob controller wakes up
2. Creates a new Job
3. Job creates Pods
4. Pods run & complete

---

## 📊 Useful Commands

```bash
kubectl get cronjobs
kubectl get jobs
kubectl get pods
kubectl describe cronjob simple-cronjob
```

---

## 🧨 Suspend a CronJob

```yaml
suspend: true
```

Stops new Job creation (existing Jobs continue).

---

## 🆚 CronJob vs Job

| Feature          | Job | CronJob |
| ---------------- | --- | ------- |
| Runs once        | ✅   | ❌       |
| Runs on schedule | ❌   | ✅       |
| Creates Jobs     | ❌   | ✅       |

---

## 🧠 Mental Model

```text
Deployment → Long-running app
Job → One-time task
CronJob → Scheduled Job creator
```

---

## 🎯 Interview One-Liner

> A CronJob schedules and automatically creates Jobs at specified time intervals using cron syntax.

---

If you want next:

* 🔥 StatefulSet notes
* 🔥 All workload comparison summary
* 🔥 Production use-case comparison table
