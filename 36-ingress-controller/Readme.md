# Ingress Controller

The Ingress controller in Kubernetes acts as a reverse proxy, which means it sits in front of web servers and acts on their behalf to forward external HTTP requests to the appropriate internal services.

The Ingress controller uses the Ingress resource to determine how to route traffic. Here's a step-by-step breakdown of how it works:

1. An external HTTP request arrives at the Ingress controller.
2. The controller examines the Ingress resource, which is defined as follows:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grade-submission-portal-ingress
  namespace: grade-submission
spec:
  ingressClassName: nginx
  rules:     
  - http:
      paths:
      - pathType: Prefix
        path: "/"    
        backend:
          service:
            name: grade-submission-portal
            port: 
              number: 5001
```

3. The controller identifies that this is an Ingress resource (`kind: Ingress`) in the `grade-submission` namespace.
4. It notes that the nginx Ingress controller should be used (`ingressClassName: nginx`).
5. The controller then looks at the rules. In this case, there's a single rule that applies to all HTTP traffic.
6. The rule specifies that all paths starting with `**/**` (`path: "/"`) should be directed to the `grade-submission-portal` service on port 5001.
7. Based on this rule, the Ingress controller forwards the request to the specified backend service.

In the current configuration, the rules are quite permissive. Any host can connect, and all traffic is routed to the same service. This setup is common in development environments but may not be suitable for production.

In a production environment, after purchasing a domain, you can implement more restrictive rules. For example:

```yaml
spec:
  rules:
  - host: grades.myuniversity.com
    http:
      paths:
      - path: "/"
        pathType: Prefix
        backend:
          service:
            name: grade-submission-portal
            port: 
              number: 5001
```

This configuration would only allow traffic from the specified host (`grades.myuniversity.com`) to be routed to the service. Any requests from other hosts would be rejected, providing an additional layer of security and control over incoming traffic.