# Extra Practice (Grade Submission API)

Grade submission API pod

<img src="../media/Grade Submission API Pod.png" style="zoom:40%;" />

<img src="../media/Grade Submission API Pod with Sidecar.png" style="zoom:40%;" />



```yaml
apiVersion: v1
kind: Pod
metadata:
  name: grade-submission-api
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

    - name: grade-submission-api-health-checker
      image: rslim087/kubernetes-course-grade-submission-api-health-checker
      resources:
        requests:
          memory: '128Mi'
          cpu: '128m'
        limits:
          memory: '128Mi'

```

>  If image tag is not specified, it's going to pull default tag `latest`



<img src="../media/Both Pods With Sidecar.png" style="zoom:40%;" />


- When we have two separate pods, they're automatically separated at the network level because of the network namespace.
- **Network Namespace** is a logical construct in Kubernetes that essentially isolates Kubernetes resources at the network level.

We want the great submission portal to forward that data to the Great Submission API, which is going to manage the submitted great data in the back end.





