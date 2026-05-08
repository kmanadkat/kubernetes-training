# Anatomy of a Connection String

Understanding the structure of basic authentication, where credentials are included directly in the URI, will be valuable in our upcoming lessons. This URI is composed of three main parts: Scheme, Auth Credentials, and Service Location.

<img src="https://img-c.udemycdn.com/redactor/raw/article_lecture/2024-07-27_00-25-10-f1e698903169c35db2288582174ae42c.png" alt="img" style="zoom:50%;" />

When a client (**grade-submission-api**) uses this URI to connect to MongoDB, here's what happens:

1. Kubernetes routing is based solely on the Service Location (`mongodb-service:27017`), regardless of the Scheme used (mongodb://, http://, https://, etc.), or the existence of credentials.
2. The ClusterIP Service `mongodb-service` receives the request at its service port `27017`.
3. The Service selects one of the MongoDB pods based on a label selector, and forwards the entire original URI to the selected pod, preserving all parts (Scheme, Auth Credentials, and Service Location).
4. MongoDB receives the full URI at its container port `27017`, and authenticates the connection using the provided credentials.

It's worth noting that this process is consistent across different types of services in Kubernetes. For example:

#### Elasticsearch

<img src="https://img-c.udemycdn.com/redactor/raw/article_lecture/2024-07-27_00-25-10-90ee5c9d58edc531c17f2e3273e749fe.png" alt="img" style="zoom:50%;" />

#### MySQL

<img src="https://img-c.udemycdn.com/redactor/raw/article_lecture/2024-07-27_00-25-10-5747d1a628c5bd445754bccc47414996.png" alt="img" style="zoom:50%;" />

In each case, Kubernetes focuses on the Service Location for routing, while the application (MySQL or Elasticsearch) handles the Scheme and Auth Credentials as needed.