[TOC]



# Kubernetes Training

**Kubernetes** is simply a tool, a software that runs across a cluster of machines. Some of these machines are master nodes and other machines are worker nodes.

- **Master nodes** are responsible for scheduling when and where your applications will run.
- **Worker nodes** are what actually run your application.
- **Cloud native app** - one that can be distributted into many micro services, each one running inside a container. 
- **Pod** - It is the smallest unit of deployment. A container cannot be deployed directly in Kubernetes, it has to be wrapperd in pod. Pod abstracts scaling, self-healing, resource allocation, etc.



## Cloud Native App Example - Grade Submission App

- **Grade submission portal** - UI to fill data and submit grade data
- **Grade submission API** - Backend processing UI submitted data



```mermaid
graph TD
    App[Grade Submission Application]
    
    subgraph PortalContainer [Container]
        Portal[Grade Submission Portal]
    end
    
    subgraph APIContainer [Container]
        API[Grade Submission API]
    end
    
    App --- PortalContainer
    App --- APIContainer

    style PortalContainer fill:#004d80,stroke:#333,color:#fff
    style APIContainer fill:#004d80,stroke:#333,color:#fff
```

<div style="padding: 40px; display: flex; justify-content: center; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;">
    <div style="width: 580px; background-color: #c0504d; padding: 15px 20px 25px 20px; border: 1px solid #333; color: white;">
        <div style="display: flex; justify-content: space-between; align-items: center; font-weight: bold; font-size: 19px; margin-bottom: 20px;">
            <span>grade-submission-portal</span>
            <span style="font-size: 14px; font-weight: normal; opacity: 0.9;">Label Groupings</span>
        </div>
        <div style="display: flex; gap: 15px; align-items: flex-start;">
            <div style="width: 320px; border: 1px solid #111;">
                <div style="background-color: #004d80; padding: 10px; text-align: center; font-weight: bold; font-size: 18px;">Container</div>
                <div style="background-color: #1a1a1a; padding: 18px; display: flex; justify-content: center;">
                    <div style="background-color: #666; padding: 10px; width: 100%; text-align: center; font-size: 15px; color: white;">Grade Submission Portal</div>
                </div>
            </div>
            <div style="display: flex; flex-direction: column; gap: 15px;">
                <div style="background-color: #00b050; padding: 4px 12px; font-weight: bold; font-size: 20px; width: fit-content; color: white;">5001</div>
                <div style="background-color: #333; padding: 10px; font-size: 14px; border: 1px solid #444; color: #ccc;">Resource Requirements</div>
            </div>
        </div>
    </div>
</div>
### Pod

A Pod has metadata and runtime requirements.

#### Metadata

- Name of the pod is usually the name of microservice.
- Labels are used to place the pod into logical groups. Labelling makes it easier to query related pods.

#### Runtime requirements

- **Port** - container port where the underlying microservice serves the requests. Pod automatically exposes this port for internal traffic.
- **Resource Requirements** - Memory and CPU - required by container. This decides which worker node to be used to run the pod.
- **memory** - the workspace where a computer stores and retrieves data for immediate use. Incompressible resource.
- **CPU** - the compute that performs calculations and executes instructions. Compressible Resource
- **requests** - minimum amount of memory and CPU that application needs to function properly.
- **limits** - prevents overaccumulation of memory. Hence saving resources in node for other pods. Since CPU is compressible, let node manage CPU allocation - more CPU if available.

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

It is recommended to prefix label keys with `app.kubernetes.io` in order to avoid potential collisions with other third party labels within the Kubernetes ecosystem.

#### Create & Retreieve Pod

```shell
# Create Pod
kubectl apply -f grade-submission-portal-pod.yaml

# Retrieve Pod
kubectl get pods

# Describe Pod
kubectl describe pod grade-submission-portal

# Streaming logs
kubectl logs -f grade-submission-portal

# Delete pod using label
kubectl delete pod -l "app.kubernetes.io/name=grade-submission"
```

#### Port Forwrading

Access the pod port where the application is serving requests on the local machine. 

```shell
kubectl port-forward grade-submission-portal 8000:5001
```

After running above, access app on localhost 8000, any requests made here will be redirected to pod port 5001.

#### Multi-Container Pod

Running two containers inside of the same pod implies that the containers need to be created and terminated at the same time. They need to be scaled up or down at the same time, these are two very separate applications and need to be scaled and created independently.

So when would you ever want to have two containers in the same pod? When the microservice relies on a sidecar to provide it with additional behavior and functionality.

<div style="padding: 40px; display: flex; justify-content: center; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;">
    <div style="width: 580px; background-color: #c0504d; padding: 15px 20px 25px 20px; border: 1px solid #333; color: white;">
        <div style="display: flex; justify-content: space-between; align-items: center; font-weight: bold; font-size: 19px; margin-bottom: 20px;">
            <span>grade-submission-portal</span>
            <span style="font-size: 14px; font-weight: normal; opacity: 0.9;">Label Groupings</span>
        </div>
        <div style="display: flex; gap: 15px;">
            <!-- Left: two containers -->
            <div style="flex: 1; display: flex; flex-direction: column; gap: 12px;">
                <!-- Container 1 -->
                <div style="border: 1px solid #111;">
                    <div style="background-color: #004d80; padding: 10px; text-align: center; font-weight: bold; font-size: 18px;">Container</div>
                    <div style="background-color: #1a1a1a; padding: 18px; display: flex; justify-content: center;">
                        <div style="background-color: #666; padding: 10px; width: 100%; text-align: center; font-size: 15px; color: white;">Grade Submission Portal</div>
                    </div>
                </div>
                <!-- Container 2 -->
                <div style="border: 1px solid #111;">
                    <div style="background-color: #004d80; padding: 10px; text-align: center; font-weight: bold; font-size: 18px;">Container</div>
                    <div style="background-color: #1a1a1a; padding: 18px; display: flex; justify-content: center;">
                        <div style="background-color: #666; padding: 10px; width: 100%; text-align: center; font-size: 15px; color: white;">Grade Submission Portal Health Checker</div>
                    </div>
                </div>
            </div>
            <!-- Right: port + line + localhost, aligned to container 1 top and container 2 center -->
            <div style="width: 80px; display: flex; flex-direction: column; align-items: center;">
                <!-- 5001 badge -->
                <div style="background-color: #00b050; padding: 4px 12px; font-weight: bold; font-size: 20px; color: white; white-space: nowrap;">5001</div>
                <!-- Line that goes down -->
                <div style="width: 2px; background-color: white; height: 100%;  flex: 1;"></div>
                <!-- localhost sits at the bottom, aligned with mid of container 2 -->
                <div style="font-size: 20px; font-weight: bold; color: white; padding-bottom: 30px;">localhost</div>
            </div>
        </div>
    </div>
</div>

Both of these containers, by virtue of running in the same pod, they're going to be scheduled to the same node.

They're going to be scheduled to the same host. They are going to share the same networking, the same network namespace, which means that these two containers can communicate with each other via localhost.

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

    # Container 2
    - name: grade-submission-portal-health-checker
      image: rslim087/kubernetes-course-grade-submission-portal-health-checker
      resources:
        requests:
          memory: '128Mi'
          cpu: '200m'
        limits:
          memory: '128Mi'

```

##### Logging

```shell
kubectl logs -f grade-submission-portal -c grade-submission-portal
kubectl logs -f grade-submission-portal -c grade-submission-portal-health-checker
```





