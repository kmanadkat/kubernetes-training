# Liveness and Readiness Probe

Extends Kubernetes self-healing beyond just detecting dead processes.

| Probe         | Problem it solves                                            | Action taken                           |
| ------------- | ------------------------------------------------------------ | -------------------------------------- |
| **Liveness**  | App is running but broken internally (deadlock, memory leak, unresponsive queue) | Restarts the container                 |
| **Readiness** | App is fine but external dependencies are down (DB, Kafka, MQ) | Blocks traffic to the pod — no restart |



### How it works

Your app exposes health endpoints that the probe hits periodically:

```
Liveness probe  → GET /health  every 5s → 200 OK = healthy, 			500 = restart container
Readiness probe → GET /ready   every 5s → 200 OK = send traffic, 	500 = block traffic
```

What the **app dev** puts in these endpoints is up to them:

- `/health` — check internal queues, caches, background tasks, disk/heap thresholds
- `/ready` — check external DB connections, Kafka, third-party APIs etc.

```yaml
spec:
  containers:
    - name: grade-submission-api
      image: rslim087/kubernetes-course-grade-submission-api
      resources:
        requests:
          memory: '128Mi'
          cpu: '200m'
        limits:
          memory: '128Mi'
      ports:
        - containerPort: 3000
      livenessProbe:
        httpGet:
          path: /health
          port: 3000
        initialDelaySeconds: 15   # wait for app to start before probing
        periodSeconds: 5          # probe every 5s after initial delay
      readinessProbe:
        httpGet:
          path: /ready
          port: 3000
        initialDelaySeconds: 10
        periodSeconds: 5
```

> ⚠️ Always set `initialDelaySeconds` on liveness probe — if the probe fires before the app has started up, it'll get a 500, restart the container, and loop forever (crash loop).

> 💡 `initialDelaySeconds` is less critical for readiness — if app isn't up yet, probe just blocks traffic (no restart). That's fine.



**Usage** Use liveness probes to detect and restart unhealthy containers. Use readiness probes to determine when a container is ready to start accepting traffic. Together, they ensure your application remains healthy and responsive in a Kubernetes environment.

