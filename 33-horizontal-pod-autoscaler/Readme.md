# Horizontal Pod Autoscaler

The Horizontal Pod Autoscaler automatically scales the number of pods in a deployment based on observed CPU utilization (most commonly).



#### Key Components of an HPA Configuration

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: grade-submission-portal-hpa
  namespace: grade-submission
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: grade-submission-portal
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

1. **Autoscaling Behavior**: The HPA will increase or decrease the number of replicas to maintain the target CPU utilization.
2. **Scaling Range**: The number of pods will be adjusted between 1 and 10 based on the CPU utilization.
3. **Resource Metrics**: While this example uses CPU, HPAs can also use memory or custom metrics.
4. **Target Utilization**: 50% target utilization is a common starting point, but this can be adjusted based on application needs.
5. **Namespace Scoping**: The HPA is namespace-specific, allowing for isolated scaling policies across different parts of your application.
6. **Scaling Algorithm**: Kubernetes uses a control loop to periodically adjust the number of replicas based on the observed metrics.

Understanding and effectively configuring HPAs is crucial for building scalable and efficient applications in Kubernetes, ensuring optimal resource utilization and performance under varying loads.

> `averageUtilization: 50` means "if average CPU across all pods exceeds 50% of their requested CPU, spin up more pods"