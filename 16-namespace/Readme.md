# Namespaces

Namespaces are **logical boundaries** used to group closely related resources in a cluster. They are not physical constructs — they consume no memory or CPU. Just invisible groupings managed by the etcd database.

> 💡 Even though a cluster is physically divided into many machines (master + worker nodes on AWS etc.), as developers we only see logical boundaries — namespaces. kubectl always feels like you're talking to one entity.



**Use cases:**

- Group related resources (e.g. all grade-submission pods + services in one namespace)
- **RBAC** — give teams access only to their namespace
- **Resource quotas** — limit memory/CPU per namespace



### Commands

```yaml
# List all namespaces
kubectl get namespaces

# Create a namespace
kubectl create namespace grade-submission

# Apply all configs in current folder to a namespace
kubectl apply -f . -n grade-submission

# Get pods/services in a specific namespace
kubectl get pods -n grade-submission
kubectl get services -n grade-submission

# Delete all pods/services in a namespace
kubectl delete pods --all -n grade-submission
kubectl delete services grade-submission-api grade-submission-portal -n default
```



#### Specifying Namespace in Config (Recommended)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: grade-submission-api
  namespace: grade-submission
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

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grade-submission-api
  namespace: grade-submission
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/instance: grade-submission-api
  ports:
    - port: 3000 # service port (internal access)
      targetPort: 3000 # pod port where app listens
```

> ⚠️ If namespace is not specified in config or command, resources go to `default` namespace automatically.



#### Best Practices for Developers

1. Use namespaces to organize and isolate your workloads logically.
2. Be aware of the namespace you're working in to avoid unintended interactions between resources.
3. Collaborate with your cluster administrators to understand any namespace-level policies or quotas that may affect your deployments.







