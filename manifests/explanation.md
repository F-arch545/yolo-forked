# Kubernetes Manifests Explanation

This document provides a detailed explanation of each Kubernetes manifest file in the `manifests/` directory and how they work together to deploy the Yolo E-commerce application.

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Namespace (01-namespace.yaml)](#namespace-01-namespaceyaml)
3. [Backend Deployment & Service (02-backend.yaml)](#backend-deployment--service-02-backendyaml)
4. [Frontend Deployment & Service (03-client.yaml)](#frontend-deployment--service-03-clientyaml)
5. [Secrets (04-secret.yaml)](#secrets-04-secretyaml)
6. [Ingress (05-ingress.yaml)](#ingress-05-ingressyaml)
7. [Deployment Order](#deployment-order)
8. [Architecture Diagram](#architecture-diagram)

---

## Overview

The manifests directory contains Kubernetes resource definitions for deploying the Yolo e-commerce platform to a Kubernetes cluster. These manifests are designed to work with AWS Elastic Kubernetes Service (EKS) using an Application Load Balancer (ALB) for external access.

### Key Technologies
- **Kubernetes**: Container orchestration
- **AWS ALB Ingress**: Load balancing
- **MongoDB Atlas**: Cloud database (connection via secret)

---

## Namespace (01-namespace.yaml)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ecommerce
```

### Purpose
Creates a logical isolation boundary for all e-commerce related resources.

### Explanation
- **`apiVersion: v1`**: Core Kubernetes API version
- **`kind: Namespace`**: Resource type for creating namespaces
- **`metadata.name: ecommerce`**: The namespace name used to organize and isolate resources

### Why Use a Namespace?
- Isolates resources for different environments (dev, staging, prod)
- Prevents naming conflicts between resources
- Enables resource quotas and access controls
- Makes resource management easier in larger clusters

---

## Backend Deployment & Service (02-backend.yaml)

This file contains two resources separated by `---`:

### Part 1: Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
  namespace: ecommerce
  labels:
    app: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: riunguflo/yolo-backend:v1.0.0
          command: ["npm", "start"]
          ports:
            - containerPort: 5000
          env:
            - name: MONGODB_URI
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: uri
          resources:
            limits:
              memory: "128Mi"
              cpu: "250m"
            requests:
              memory: "64Mi"
              cpu: "100m"
```

#### Deployment Explanation

| Field | Value | Description |
|-------|-------|-------------|
| `apiVersion` | `apps/v1` | API version for Deployment resources |
| `kind` | `Deployment` | Creates a deployment controller |
| `replicas` | `1` | Number of pod instances |
| `image` | `riunguflo/yolo-backend:v1.0.0` | Docker image from registry |
| `command` | `["npm", "start"]` | Command to start the application |
| `containerPort` | `5000` | Port the container listens on |

#### Resource Limits
- **Limits**: Maximum resources the container can use
  - Memory: 128 MiB
  - CPU: 250m (millicores)
- **Requests**: Minimum resources guaranteed
  - Memory: 64 MiB
  - CPU: 100m

#### Environment Variables
- `MONGODB_URI`: Sourced from the `mongo-secret` Kubernetes Secret
- Uses `secretKeyRef` for secure credential management

### Part 2: Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: ecommerce
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
```

#### Service Explanation

| Field | Value | Description |
|-------|-------|-------------|
| `type` | `ClusterIP` | Internal cluster IP (not exposed externally) |
| `selector` | `app: backend` | Selects pods with this label |
| `port` | `5000` | Port the service listens on |
| `targetPort` | `5000` | Port on the pod to forward traffic to |

#### Why ClusterIP?
The backend service is internal because:
- Only the frontend needs to communicate with it
- The Ingress controller routes external traffic to it
- Security: Not directly accessible from outside the cluster

---

## Frontend Deployment & Service (03-client.yaml)

This file also contains two resources separated by `---`:

### Part 1: Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  namespace: ecommerce
  labels:
    app: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: riunguflo/yolo-client:v1.0.0
          ports:
            - containerPort: 80
          env:
            - name: REACT_APP_API_URL
              value: "https://yourdomain.com/api"
          resources:
            limits:
              memory: "128Mi"
              cpu: "250m"
            requests:
              memory: "64Mi"
              cpu: "100m"
```

#### Frontend Container Details

| Field | Value | Description |
|-------|-------|-------------|
| `image` | `riunguflo/yolo-client:v1.0.0` | React app built with Nginx |
| `containerPort` | `80` | Nginx listens on port 80 |
| `REACT_APP_API_URL` | `https://yourdomain.com/api` | Backend API URL (injected at runtime) |

#### React Environment Variables
The frontend uses Create React App's environment variable pattern:
- Variables prefixed with `REACT_APP_` are embedded at build time
- `REACT_APP_API_URL` tells the frontend where to make API calls

### Part 2: Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: ecommerce
spec:
  type: ClusterIP
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

The frontend service is also ClusterIP because:
- The Ingress controller handles external access
- Provides stable internal endpoint for the ALB to route to

---

## Secrets (04-secret.yaml)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
  namespace: ecommerce
type: Opaque
stringData:
  uri: "mongodb+srv://flozy:flozy@cluster1.ksklg53.mongodb.net/ecommerce?retryWrites=true&w=majority"
```

### Secret Explanation

| Field | Value | Description |
|-------|-------|-------------|
| `type` | `Opaque` | Default secret type for arbitrary key-value pairs |
| `stringData` | - | Plaintext data that will be encoded |
| `uri` | MongoDB connection string | Contains username, password, and host |

#### Security Best Practices

⚠️ **Important**: This is a demo configuration with plaintext credentials. For production:

1. **Use sealed secrets** or **Vault** for secret management
2. **Never commit secrets to version control**
3. **Use IAM roles** for AWS RDS/DocumentDB authentication
4. **Rotate credentials regularly**

#### Alternative: External Secrets Operator
For production on AWS, consider using:
- [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)
- [External Secrets Operator](https://external-secrets.io/)

---

## Ingress (05-ingress.yaml)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  namespace: ecommerce
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:123456789012:certificate/your-cert-id
    alb.ingress.kubernetes.io/ssl-redirect: "443"
spec:
  ingressClassName: alb
  rules:
    - host: yourdomain.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 5000
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

### Ingress Explanation

#### AWS ALB Ingress Controller

This ingress is configured for AWS Application Load Balancer (ALB) using the [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/).

#### Annotations

| Annotation | Value | Purpose |
|------------|-------|---------|
| `scheme` | `internet-facing` | ALB is publicly accessible |
| `target-type` | `ip` | Direct pod IP routing |
| `listen-ports` | `[{"HTTP":80},{"HTTPS":443}]` | Support both HTTP and HTTPS |
| `certificate-arn` | ACM certificate ARN | SSL/TLS certificate for HTTPS |
| `ssl-redirect` | `"443"` | Redirect HTTP to HTTPS |

#### Path-Based Routing

```
                    ┌─────────────────────────────────────────┐
                    │            AWS Application              │
                    │            Load Balancer (ALB)          │
                    └─────────────────────────────────────────┘
                                     │
              ┌──────────────────────┼──────────────────────┐
              │                      │                      │
              ▼                      ▼                      ▼
         /api/*              /* (everything else)           │
              │                      │                      │
              ▼                      ▼                      ▼
    ┌─────────────────┐    ┌─────────────────┐             │
    │  Backend        │    │  Frontend       │             │
    │  Service:5000   │    │  Service:80     │             │
    └─────────────────┘    └─────────────────┘             │
              │                      │                      │
              ▼                      ▼                      ▼
    ┌─────────────────┐    ┌─────────────────┐             │
    │  Backend Pod    │    │  Frontend Pod   │             │
    │  (Port 5000)    │    │  (Port 80)      │             │
    └─────────────────┘    └─────────────────┘             │
```

### Rule Breakdown

| Path | PathType | Service | Service Port | Description |
|------|----------|---------|--------------|-------------|
| `/api` | Prefix | backend-service | 5000 | Routes API requests to backend |
| `/` | Prefix | frontend-service | 80 | Routes all other requests to frontend |

#### Why PathType: Prefix?
- `/api` with `Prefix` matches any path starting with `/api`
- `/` with `Prefix` matches everything else
- This ensures the frontend receives all non-API requests (SPA routing)

---

## Deployment Order

Apply manifests in this order:

```bash
# 1. Create namespace
kubectl apply -f 01-namespace.yaml

# 2. Create secret (required by backend)
kubectl apply -f 04-secret.yaml

# 3. Create deployments and services
kubectl apply -f 02-backend.yaml
kubectl apply -f 03-client.yaml

# 4. Create ingress (requires ALB Controller)
kubectl apply -f 05-ingress.yaml
```

Or apply all at once:
```bash
kubectl apply -f manifests/
```

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Kubernetes Cluster                          │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   ecommerce Namespace                       │   │
│  │                                                             │   │
│  │  ┌─────────────────────┐    ┌─────────────────────────┐    │   │
│  │  │   Ingress           │    │   Secrets               │    │   │
│  │  │   (ALB)             │    │   mongo-secret          │    │   │
│  │  └─────────┬───────────┘    └─────────────────────────┘    │   │
│  │            │                                                 │   │
│  │  ┌─────────┴─────────┐                                       │   │
│  │  │                   │                                       │   │
│  │  ▼                   ▼                                       │   │
│  │  ┌───────────────────────┐    ┌──────────────────────┐      │   │
│  │  │   frontend-service    │    │   backend-service    │      │   │
│  │  │   (ClusterIP:80)      │    │   (ClusterIP:5000)   │      │   │
│  │  └───────────┬───────────┘    └──────────┬───────────┘      │   │
│  │              │                           │                  │   │
│  │              ▼                           ▼                  │   │
│  │  ┌───────────────────────┐    ┌──────────────────────┐      │   │
│  │  │   frontend-deployment │    │   backend-deployment │      │   │
│  │  │   Pod: frontend       │    │   Pod: backend       │      │   │
│  │  │   Container: nginx:80 │    │   Container: node:5000│     │   │
│  │  └───────────────────────┘    └───────────┬──────────┘      │   │
│  │                                           │                  │   │
│  └───────────────────────────────────────────┼──────────────────┘   │
│                                              │                      │
└──────────────────────────────────────────────┼──────────────────────┘
                                               │
                                               ▼
                                  ┌─────────────────────────┐
                                  │   MongoDB Atlas         │
                                  │   (External Service)    │
                                  │   mongodb+srv://...     │
                                  └─────────────────────────┘
```

---

## Prerequisites

Before deploying, ensure you have:

1. **AWS EKS Cluster** with:
   - AWS Load Balancer Controller installed
   - IAM roles for service accounts (IRSA) configured

2. **ACM Certificate**:
   - Request a certificate in AWS Certificate Manager
   - Validate domain ownership
   - Note the certificate ARN

3. **kubectl** configured with cluster access:
   ```bash
   aws eks update-kubeconfig --name your-eks-cluster
   ```

---

## Verification

After deployment, verify resources:

```bash
# Check namespace
kubectl get ns ecommerce

# Check all resources in namespace
kubectl get all,secret,ingress -n ecommerce

# Check ingress status
kubectl describe ingress ecommerce-ingress -n ecommerce

# View pod logs
kubectl logs -f deployment/backend-deployment -n ecommerce
kubectl logs -f deployment/frontend-deployment -n ecommerce
```

---

## Troubleshooting

### Pods not starting
```bash
kubectl describe pod <pod-name> -n ecommerce
kubectl logs <pod-name> -n ecommerce
```

### ImagePullBackOff
- Verify Docker images exist: `riunguflo/yolo-backend:v1.0.0` and `riunguflo/yolo-client:v1.0.0`
- Check image pull permissions

### ALB not created
- Verify AWS Load Balancer Controller is running:
  ```bash
  kubectl get pods -n kube-system | grep load-balancer
  ```
- Check controller logs:
  ```bash
  kubectl logs -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
  ```

---

## Additional Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/)
- [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

