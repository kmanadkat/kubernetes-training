# Database Authentication

By default MongoDB allows any service to connect without credentials. Pass environment variables to enable authentication — MongoDB reads these on startup to enforce username/password access.

### MongoDB — Enable Authentication via Env Vars

```yaml
# mongodb-statefulset.yaml
containers:
  - name: mongodb
    image: mongo:4.4
    ports:
      - containerPort: 27017
    env:
      - name: MONGO_INITDB_ROOT_USERNAME
        value: admin
      - name: MONGO_INITDB_ROOT_PASSWORD
        value: password123        # ⚠️ hardcoding passwords in YAML is bad — see ConfigMap/Secret section
    volumeMounts:
      - name: mongodb-persistent-storage
        mountPath: /data/db
```

> 💡 If these env vars are not provided, MongoDB assumes no authentication is needed and allows open access.



### API — Pass Credentials in Connection String

App needs 4 env vars to connect to an auth-enabled MongoDB:

yaml

```yaml
# grade-submission-api-deployment.yaml
 spec:
      containers:
        - name: grade-submission-api
          image: rslim087/kubernetes-course-grade-submission-api:stateless-v3
          env:
            - name: MONGODB_HOST
              value: mongodb # Name of the service
            - name: MONGODB_PORT
              value: "27017" # Service Port
            - name: MONGODB_USER
              value: admin
            - name: MONGODB_PASSWORD
              value: password123
```

```shell
API container → connection string → ClusterIP Service (mongodb:27017)
                                          ↓
                                    MongoDB checks username + password
                                          ↓
                              ✅ match → connected   ❌ mismatch → rejected
```

> 💡 Always check the image docs for the exact env var names — they differ per database (MySQL, PostgreSQL, MongoDB etc.)



### Debugging Auth Failures

bash

```bash
# Check if pods are stuck at 0/1 (readiness probe failing = can't connect to DB)
kubectl get pods -n grade-submission

# Stream logs to see auth errors
kubectl logs -f <api-pod-name> -n grade-submission
# Authentication failed  ← wrong password in connection string
```

------

### ⚠️ Problem with this approach

Hardcoding passwords directly in YAML manifests is bad:

- Special characters in passwords break YAML syntax
- Credentials mixed with deployment config = hard to maintain
- Not environment-specific (dev vs prod passwords differ)

**Solution → ConfigMap & Secret** (next section)