# Service Discovery

Port forwarding isn't a very reliable way to access our application. We need something more stable, something more durable.

<img src="../media/Grade Submission Portal Pod.png" style="zoom:40%;" />


- Every pod in Kubernetes is assigned a **virtual IP address**. 
- However, pods are **ephemeral** — constantly created and destroyed for scaling/self-healing — and each recreation gives them a **new IP address**. So pod IPs are unreliable for access.
- **Services** solve this by providing stable endpoints to access pods using **label selectors** (not IPs).



### NodePort Service

Used for **external access** to pods. Opens a static port (30000–32000) on the node so the app can be reached from outside the cluster.

```
Browser → localhost:32000 (NodePort) → label selector → Pod:5001
```

#### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grade-submission-portal
spec:
  type: NodePort
  selector:
    app.kubernetes.io/instance: grade-submission-portal
  ports:
    - port: 5001             # service port (internal access)
      targetPort: 5001       # pod port where app listens
      nodePort: 32000        # external port on the node (30000-32000)
```

#### Pod

```yaml
apiVersion: v1
kind: Pod

# Metadata
metadata:
  name: grade-submission-portal
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
      resources:
        requests:
          memory: '128Mi'
          cpu: '200m'
        limits:
          memory: '128Mi'
      ports:
        - containerPort: 5001

```

#### Diagram

<img src="../media/NodePort Service & Pod.png" style="zoom:40%;" />


> ⚠️ **Never use NodePort in production.** It opens a port on *every* worker node, even ones not running your app — bad for security and scaling. Use it only for local prototyping (single-node cluster).

<img src="../media/NodePort - Worker Nodes.png" style="zoom:40%;" />

### ClusterIP Service

Used for **internal pod-to-pod communication**. Gets a cluster-internal IP, but you don't need to know it — the **service name resolves to it automatically** within the cluster.

Below `grade-submission-api` is the name of ClusterIP service. Port `3000` is the port `portal` pod has decided to send the request. This port need not be same as `containerPort` of API service pod.

```
Portal Pod → http://grade-submission-api:3000 → ClusterIP Service → label selector → API Pod:3000
```

We want the service port and the target port to match, because from the application's perspective, it doesn't know that it's making requests to a cluster IP service. This looks like it's making requests directly to the Great Submission API at its normal port.

<img src="../media/NodePort ClusterIP & Pods.png" style="zoom:40%;" />

#### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grade-submission-api
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/instance: grade-submission-api
  ports:
    - port: 3000 # service port (internal access)
      targetPort: 3000 # pod port where app listens
```

#### Pod

```yaml
apiVersion: v1
kind: Pod

# Metadata
metadata:
  name: grade-submission-portal
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























