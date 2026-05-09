# ConfigMap & Secret

Separate app-specific configuration from deployment details. Keeps manifests clean, maintainable, and environment-flexible.

|           | ConfigMap                 | Secret                               |
| --------- | ------------------------- | ------------------------------------ |
| Data type | Non-sensitive config      | Sensitive config (passwords, tokens) |
| Storage   | Plain text                | Base64 encoded                       |
| Use case  | Host, port, service names | DB credentials, API keys             |



### Why separate config from deployment?

- Updating a password shouldn't require editing a StatefulSet
- 10+ env vars flooding a deployment YAML = unmaintainable
- Dev vs prod configs differ — separation makes swapping easy

<img src="../media/ConfigMap & Secrets.png" style="zoom:40%;" />

#### Configmap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grade-submission-api-config
  namespace: grade-submission  					# must match deployment namespace
data:
  MONGODB_HOST: 'mongodb' 							# plain text key-value pairs
  MONGODB_PORT: '27017'
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grade-submission-portal-config
  namespace: grade-submission
data:
  GRADE_SERVICE_HOST: 'grade-submission-api' # resolves to the ClusterIP service - value is name of service.
```



### Secret

Values must be **base64 encoded** — Kubernetes decodes back to plaintext at container runtime.

bash

```bash
# Encode values before putting in secret
echo -n "admin" | base64        # → YWRtaW4=
echo -n "password123" | base64  # → cGFzc3dvcmQxMjM=
```

> 💡 Base64 is NOT encryption — it just converts any string (including special characters) into 64 safe ASCII characters so it can safely live in YAML. Secrets are about separation, not security by itself.



#### Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: grade-submission-api-secret
  namespace: grade-submission
type: Opaque
data:
  MONGODB_USER: 'YWRtaW4=' # admin in base64
  MONGODB_PASSWORD: 'cGFzc3dvcmQxMjM=' # password123 in base64

```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
  namespace: grade-submission
type: Opaque
data:
  MONGO_INITDB_ROOT_USERNAME: 'YWRtaW4='
  MONGO_INITDB_ROOT_PASSWORD: 'cGFzc3dvcmQxMjM='
```



### Referencing in Deployments

Instead of hardcoding env vars, reference the ConfigMap and Secret:

Instead of hardcoding env vars, reference the ConfigMap and Secret:

```yaml
spec:
  containers:
    - name: grade-submission-api
      image: rslim087/kubernetes-course-grade-submission-api-stateless-v3
      envFrom:
        - configMapRef:
            name: grade-submission-api-config    # loads all keys from configmap
        - secretRef:
            name: grade-submission-api-secret    # loads all keys from secret
```





