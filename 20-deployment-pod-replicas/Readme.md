# Deployments

Instead of managing standalone pods, use a **Deployment** — it manages pod replicas for you and ensures the desired number of healthy pods is always running.

```
Deployment → creates → ReplicaSet → creates & monitors → N x Pods
```

**Benefits over standalone pods:**

- Auto-creates specified number of pod replicas
- Continuously monitors pods — if one dies, recreates it to match desired state
- ClusterIP services distribute traffic across replicas (**load balancing out of the box**)

#### API Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grade-submission-api
  namespace: grade-submission
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/instance: grade-submission-api
  template:
    metadata:
      labels:
        app.kubernetes.io/name: grade-submission
        app.kubernetes.io/component: backend
        app.kubernetes.io/instance: grade-submission-api
    spec:
      containers:
        - name: grade-submission-api
          image: rslim087/kubernetes-course-grade-submission-api:stateless
          resources:
            requests:
              memory: '128Mi'
              cpu: '128m'
            limits:
              memory: '128Mi'
          ports:
            - containerPort: 3000
```

#### Portal Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grade-submission-portal
  namespace: grade-submission
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/instance: grade-submission-portal
  template:
    metadata:
      labels:
        app.kubernetes.io/name: grade-submission
        app.kubernetes.io/component: frontend
        app.kubernetes.io/instance: grade-submission-portal

    # Runtime Requirements
    spec:
      containers:
        # Container 1
        - name: grade-submission-portal
          image: rslim087/kubernetes-course-grade-submission-portal
          env:
            - name: GRADE_SERVICE_HOST
              value: grade-submission-api # resolves to the ClusterIP service - value is name of service.
          resources:
            requests:
              memory: '128Mi'
              cpu: '200m'
            limits:
              memory: '128Mi'
          ports:
            - containerPort: 5001
```

### How Services + Deployments Work Together

The ClusterIP service forwards traffic to **any pod matching its label selector** — so it automatically load balances across all replicas in round-robin.

<img src="../media/NodePort ClusterIP Load Balancer.png" style="zoom:40%;" />

> 💡 Always manage pods through Deployments, never standalone pod primitives. Even for a single pod — if it dies, a deployment will recreate it automatically.

<img src="../media/Automated Deployment.png" style="zoom:40%;" />