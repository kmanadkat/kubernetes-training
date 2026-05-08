# Rolling Updates and Rollback

Without rolling updates, upgrading an application could lead to significant downtime. Imagine a scenario where all pod replicas running v1 of an application are simultaneously taken down and replaced with v2. This would result in a period where no pods are available to serve traffic, causing service interruption. Rolling updates solve this problem by gradually replacing old pods with new ones, ensuring that the application remains available throughout the upgrade process.

### How Rolling Updates Work

**1.** The process begins when you update the pod template in a Deployment resource. Here's an example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grade-submission-api
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: grade-submission-api
        image: registry/grade-submission-api:v1 # Update to v2
```

**2.** The Deployment controller creates a new ReplicaSet for v2 pods while preserving the old ReplicaSet for v1 pods. A ReplicaSet is a Kubernetes resource that ensures a specified number of pod replicas are running at all times.

During a rolling update, the Deployment manages two ReplicaSets:

```shell
   ReplicaSet 1 (v1): 3 pods ----> 0 pods
   ReplicaSet 2 (v2): 0 pods ----> 3 pods
```

**3.** New v2 pods are created while old v1 pods are gradually terminated. This process continues until all pods are running the new version.



### Controlling the Rolling Update

By default, Kubernetes implements a rolling update strategy, but you can control its behavior using two parameters:

1. `maxUnavailable`: The maximum number of pods that can be unavailable during the update.
2. `maxSurge`: The maximum number of pods that can be created over the desired number of pods.

Here's how you can specify these in your Deployment:

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

With these settings and 3 replicas, you might see output like this during the update:

```shell
NAME                                   READY   STATUS
grade-submission-api-7f9dd7f789-2x9p4   1/1     Running
grade-submission-api-7f9dd7f789-9k2xn   1/1     Running
grade-submission-api-7f9dd7f789-qp6g7   1/1     Running
grade-submission-api-6c54bd5869-lmn0p   0/1     ContainerCreating
```

This output reflects our settings:

- All three original pods are running, respecting `maxUnavailable: 1`.
- One new pod is being created, adhering to `maxSurge: 1`.

As the update progresses:

```shell
NAME                                   READY   STATUS
grade-submission-api-7f9dd7f789-2x9p4   1/1     Running
grade-submission-api-7f9dd7f789-9k2xn   1/1     Running
grade-submission-api-7f9dd7f789-qp6g7   0/1     Terminating
grade-submission-api-6c54bd5869-lmn0p   1/1     Running
grade-submission-api-6c54bd5869-3fgh2   1/1     Running
```

Here, we see:

- One old pod terminating (`maxUnavailable: 1`)
- Two new pods running, temporarily giving us 4 pods total (`maxSurge: 1`)
- At least 2 pods always fully running, ensuring minimal disruption

This process continues until all pods are updated to the new version.



### Benefits of Rolling Updates

**1. Minimal Downtime**: By gradually replacing pods, the application remains available throughout the update process.

**2. Version Control**: Easy rollbacks to previous versions if issues are detected.

**3. Traffic Management**: Ensures that user traffic is always directed to available pods.



## Rollbacks

If issues arise with the new version, Kubernetes makes rollbacks easy. The old ReplicaSet (v1) is kept, allowing for quick reversion. During a rollback:

1. The v2 ReplicaSet is scaled down to 0 pods.
2. A new ReplicaSet based on the v1 configuration is created and scaled up to the desired number of replicas.
3. This process also uses a rolling update strategy, ensuring minimal downtime during the rollback.

```shell
ReplicaSet 1 (v1): 3 pods ----> 0 pods
ReplicaSet 2 (v2): 3 pods ----> 0 pods
ReplicaSet 3 (v1): 0 pods ----> 3 pods
```

```shell
# Roll back to previous version
kubectl rollout undo deployment grade-submission-api -n grade-submission

# Watch gradual rollback
kubectl get pods -n grade-submission
```



### Graceful Shutdown (in-flight requests during rollout)

When Kubernetes terminates an old pod, it sends **SIGTERM** (polite stop) and waits for a grace period before force-killing with **SIGKILL**. In-flight requests can finish within this window.

```
t=0s  → API call received, backend starts processing (takes 30s)
t=5s  → Rolling update kicks in, Kubernetes sends SIGTERM to old pod
t=35s → Request finishes ✅ pod shuts down cleanly (within grace period)
t=35s → If request hadn't finished → SIGKILL fires → caller gets error ❌
```

Default grace period is **30 seconds**. Tune it if your requests take longer:

yaml

```yaml
spec:
  terminationGracePeriodSeconds: 60  # default 30 — set higher than longest request duration
  containers:
    - name: grade-submission-api
      ...
```

> ⚠️ Graceful shutdown only works if your **app handles SIGTERM** — i.e. stops accepting new requests but finishes existing ones. Most modern frameworks (Express, Spring Boot, FastAPI) support this out of the box or with one config flag.