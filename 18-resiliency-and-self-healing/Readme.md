## Self-Healing & Resiliency

Kubernetes automatically restarts containers when they fail — keeping apps available and minimizing downtime.

---

### Restart Policy

Default restart policy for every container is `always` — no need to write it explicitly.

| Policy | Behaviour |
|---|---|
| `Always` (default) | Restart container whenever process terminates, regardless of exit code |
| `OnFailure` | Restart only on non-zero exit code |
| `Never` | Never restart |

```yaml
spec:
  restartPolicy: Always   # default — not required to specify
  containers:
    - name: grade-submission-api
      image: rslim087/kubernetes-course-grade-submission-api
      resources:
        requests:
          memory: '35Mi'
          cpu: '200m'
        limits:
          memory: '35Mi'   # 00MKilled if exceeded — triggers restart
```

---

### 00MKilled — Memory Limit Exceeded

Memory is **incompressible** — it can't be throttled. When a container exceeds its memory limit, Kubernetes **terminates the process** (OOMKilled). The restart policy then brings it back.

```
container exceeds memory limit
        ↓
process terminated (00MKilled)
        ↓
restart policy = Always
        ↓
container restarted → healthy again
```

```bash
# Check pod status + restart count
kubectl get pods -n grade-submission

# NAME                    READY   STATUS    RESTARTS   AGE
# grade-submission-api    1/1     Running   1          5m   ← restarted once
```

> 💡 `RESTARTS` column in `kubectl get pods` tells you how many times a container has been restarted.

---

### Port Forwarding (for local testing)

```bash
# Forward pod port to local machine for testing
kubectl port-forward grade-submission-api 9090:3000 -n grade-submission
# localhost:9090 → pod:3000
```

> ⚠️ When a container dies and restarts, active port-forwards break. Re-run the command after restart.

---

### Key Takeaway

Kubernetes doesn't prevent failures — it **recovers from them automatically**. Any failure that terminates a container process (OOMKill, app crash, bad exit code) will trigger a restart based on the restart policy.
