# StatefulSet & Persistent Volumes

For **stateful apps** (databases) that need durable storage. Contrast with Deployments which manage stateless pods.

|                       | Deployment                            | StatefulSet                               |
| --------------------- | ------------------------------------- | ----------------------------------------- |
| Pod names             | random suffix (`api-7d6b9c8f4-xk2pq`) | ordered suffix (`mongodb-0`, `mongodb-1`) |
| Pods interchangeable? | ✅ yes                                 | ❌ no — each has unique identity           |
| Storage               | shared or none                        | each pod gets its own PVC                 |
| Scaling order         | unordered                             | ordered (0 first, then 1, 2...)           |
| Use case              | APIs, frontends                       | Databases, message brokers                |



### How Storage Works

```
StatefulSet Pod (mongodb-0)
    └── bound to PVC (mongodb-persistent-storage-mongodb-0)
            └── bound to PV (actual physical storage on a node)
                    └── mounted at /data/db inside container
```

If `mongodb-0` is deleted → rescheduled with **same name** → reattaches to **same PVC** → data intact ✅



<img src="../media/Statefulset Pod.png" style="zoom:40%;" />



#### mongodb-statefulset

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
  namespace: grade-submission
spec:
  selector:
    matchLabels:
      app.kubernetes.io/instance: mongodb
  serviceName: mongodb																# prefix for pod names → mongodb-0, mongodb-1
  replicas: 1 																				# use 1 for a single DB — multiple causes split data
  template:
    metadata:
      labels:
        app.kubernetes.io/name: grade-submission
        app.kubernetes.io/component: database
        app.kubernetes.io/instance: mongodb
    spec:
      containers:
        - name: mongodb
          image: mongo:4.4
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: mongodb-persistant-storage
              mountPath: /data/db											# where MongoDB saves data inside container
  volumeClaimTemplates:																# generates one PVC per pod replica
    - metadata:
        name: mongodb-persistant-storage
      spec:
        accessModes: [ "ReadWriteOnce" ]							# only one node can read/write at a time
        resources:
          requests:
            storage: 1Gi
```



#### Cluster IP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb
  namespace: grade-submission
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/instance: mongodb
  ports:
    - port: 27017
      targetPort: 27017

```



<img src="../media/Statefulset and ClusterIP.png" style="zoom:40%;" />

#### POD Connection to Mongodb

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
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 50%
      maxSurge: 1
  template:
    metadata:
      labels:
        app.kubernetes.io/name: grade-submission
        app.kubernetes.io/component: backend
        app.kubernetes.io/instance: grade-submission-api
    spec:
      containers:
        - name: grade-submission-api
          image: rslim087/kubernetes-course-grade-submission-api:stateless-v2
          env:
            - name: MONGODB_HOST
              value: mongodb # Name of the service
            - name: MONGODB_PORT
              value: "27017" # Service Port
          livenessProbe:
            httpGet:
              path: /healthz
              port: 3000
            initialDelaySeconds: 15 # wait for app to start before probing
            periodSeconds: 5 # probe every 5s after initial delay
          readinessProbe:
            httpGet:
              path: /readyz
              port: 3000
            periodSeconds: 5 # probe every 5s
          resources:
            requests:
              memory: '128Mi'
              cpu: '128m'
            limits:
              memory: '128Mi'
          ports:
            - containerPort: 3000

```

> 💡 Readiness probe is critical here — API pod shows `0/1` until it successfully connects to MongoDB, then flips to `1/1`. Without it you'd be sending traffic to an API that can't talk to DB yet.



#### ⚠️ Why replicas: 1 for MongoDB here

With 2 replicas, the ClusterIP service round-robins between `mongodb-0` and `mongodb-1`. Each has **separate storage** → different data → inconsistent reads.

Real production MongoDB clustering requires a **Replica Set** with leader election — that's a separate setup, not just increasing `replicas`.



#### What is mounting?

When you attach an EBS volume to EC2, AWS asks you — **where in the filesystem should this storage appear?** That's mounting.

```
EBS Volume (physical storage somewhere in AWS)
    └── mounted at /data on your EC2 instance
            └── now ls /data shows files stored on that EBS
```

Without mounting, the storage exists but your OS has no idea where to find it. Mounting is just **pointing a directory to a storage device**.



#### Same concept in Kubernetes

```
Persistent Volume (physical storage on a node)
    └── claimed by PVC
        └── mounted at /data/db inside the container
                └── MongoDB writes here thinking it's local disk
                    but actually writing to durable external storage
```

The `volumeMounts` in your YAML is just saying — **"take that PV and make it appear at this path inside my container"**.

```yaml
volumeMounts:
  - name: mongodb-persistent-storage
    mountPath: /data/db   # MongoDB thinks this is local — it's actually the PV
```











