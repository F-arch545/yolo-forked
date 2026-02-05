# Kubernetes Manifests for Yolo E-Commerce

This directory contains Kubernetes manifest files for deploying the Yolo E-commerce application to a Kubernetes cluster.

---

## 📁 Manifest Files

| File | Description |
|------|-------------|
| `01-namespace.yaml` | Creates the `ecommerce` namespace |
| `02-backend.yaml` | Backend Deployment and Service (Node.js API) |
| `03-client.yaml` | Frontend Deployment and Service (React/Nginx) |
| `04-secret.yaml` | MongoDB connection secret |
| `05-ingress.yaml` | AWS ALB Ingress for external access |

---

## 🌐 Live Deployment (EC2)

The Yolo E-Commerce application is deployed on AWS EC2 with Kubernetes and accessible via:

**Live URL**: [http://a2f94f894be434d4c8f57340f59bc977-1852385997.us-east-1.elb.amazonaws.com](http://a2f94f894be434d4c8f57340f59bc977-1852385997.us-east-1.elb.amazonaws.com)

---

## 🚀 Quick Deployment

### Prerequisites

- Kubernetes cluster (AWS EKS recommended)
- `kubectl` configured with cluster access
- AWS Load Balancer Controller installed
- ACM certificate for HTTPS

### Deploy All Manifests

```bash
# Apply all manifests
kubectl apply -f manifests/

# Or apply individually in order
kubectl apply -f 01-namespace.yaml
kubectl apply -f 04-secret.yaml
kubectl apply -f 02-backend.yaml
kubectl apply -f 03-client.yaml
kubectl apply -f 05-ingress.yaml
```

### Verify Deployment

```bash
# Check all resources
kubectl get all,secret,ingress -n ecommerce

# Check pod status
kubectl get pods -n ecommerce

# View logs
kubectl logs -f deployment/backend-deployment -n ecommerce
kubectl logs -f deployment/frontend-deployment -n ecommerce
```

---

## 🏗️ Architecture

```
Internet
    │
    ▼
┌─────────────────────────────────────────┐
│  AWS Application Load Balancer (ALB)    │
│  - HTTPS (port 443)                     │
│  - HTTP redirect to HTTPS               │
│  - Path-based routing                   │
└────────────────┬────────────────────────┘
                 │
    ┌────────────┼────────────┐
    │            │            │
    ▼            ▼            ▼
/api/*          /           (static)
/api/products   │
    │           │
    ▼           ▼
┌────────┐  ┌─────────────────┐
│Backend │  │    Frontend     │
│Service │  │    Service      │
│ :5000  │  │    :80          │
└────┬───┘  └────────┬────────┘
     │               │
     ▼               ▼
┌─────────┐    ┌─────────────┐
│ Backend │    │  Frontend   │
│  Pod    │    │    Pod      │
│ :5000   │    │    :80      │
└─────────┘    └─────────────┘
     │
     ▼
┌─────────────────────────┐
│    MongoDB Atlas        │
│  (External Database)    │
└─────────────────────────┘
```

---

## 📋 Resources Created

### Namespace
- `ecommerce` - Logical isolation for all resources

### Deployments
| Name | Container | Image | Port | Replicas |
|------|-----------|-------|------|----------|
| `backend-deployment` | backend | `riunguflo/yolo-backend:v1.0.0` | 5000 | 1 |
| `frontend-deployment` | frontend | `riunguflo/yolo-client:v1.0.0` | 80 | 1 |

### Services
| Name | Type | Selector | Port |
|------|------|----------|------|
| `backend-service` | ClusterIP | app: backend | 5000 → 5000 |
| `frontend-service` | ClusterIP | app: frontend | 80 → 80 |

### Ingress
| Name | Class | Host | Paths |
|------|-------|------|-------|
| `ecommerce-ingress` | alb | `yourdomain.com` | `/api` → backend:5000, `/` → frontend:80 |

### Secrets
| Name | Type | Data |
|------|------|------|
| `mongo-secret` | Opaque | `uri` - MongoDB Atlas connection string |

---

## ⚙️ Configuration

### Update Domain Name

Edit `05-ingress.yaml` and replace:
```yaml
rules:
  - host: yourdomain.com  # Change to your domain
```

### Update ACM Certificate ARN

Edit `05-ingress.yaml`:
```yaml
annotations:
  alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:123456789012:certificate/your-cert-id
```

### Update Backend API URL

Edit `03-client.yaml`:
```yaml
env:
  - name: REACT_APP_API_URL
    value: "https://yourdomain.com/api"  # Your actual domain
```

### Update MongoDB Connection

Edit `04-secret.yaml`:
```yaml
stringData:
  uri: "mongodb+srv://username:password@cluster.mongodb.net/ecommerce"
```

---

## 🔒 Security Considerations

⚠️ **Important**: The current configuration includes plaintext credentials in `04-secret.yaml`.

For production deployment:

1. **Use sealed secrets** or **external secrets**:
   ```bash
   # Install External Secrets Operator
   helm repo add external-secrets https://charts.external-secrets.io
   helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace
   ```

2. **Store secrets in AWS Secrets Manager**:
   ```yaml
   apiVersion: external-secrets.io/v1beta1
   kind: ExternalSecret
   metadata:
     name: mongo-secret
   spec:
     refreshInterval: 1h
     secretStoreRef:
       name: aws-secrets-manager
       type: aws
     target:
       name: mongo-secret
     data:
       - secretKey: uri
         remoteRef:
           key: prod/mongodb/atlas
           property: connection-string
   ```

3. **Never commit secrets to version control**

---

## 🔧 Resource Management

### Update Resources

```bash
# Update a deployment
kubectl set image deployment/backend-deployment backend=riunguflo/yolo-backend:v1.1.0 -n ecommerce

# View rollout status
kubectl rollout status deployment/backend-deployment -n ecommerce

# Rollback if needed
kubectl rollout undo deployment/backend-deployment -n ecommerce
```

### Scale Resources

```bash
# Scale backend
kubectl scale deployment backend-deployment --replicas=3 -n ecommerce

# Scale frontend
kubectl scale deployment frontend-deployment --replicas=2 -n ecommerce
```

---

## 🧹 Cleanup

```bash
# Delete all resources
kubectl delete -f manifests/

# Or delete individually
kubectl delete ingress ecommerce-ingress -n ecommerce
kubectl delete -f 03-client.yaml
kubectl delete -f 02-backend.yaml
kubectl delete secret mongo-secret -n ecommerce
kubectl delete namespace ecommerce
```

---

## 📖 Additional Documentation

- **[Explanation](explanation.md)** - Detailed explanation of each manifest
- **[Docker Hub Images](https://hub.docker.com/u/riunguflo)** - Backend: `riunguflo/yolo-backend`, Client: `riunguflo/yolo-client`
- **[AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/)**

---

## 🆘 Troubleshooting

### Pods in CrashLoopBackOff
```bash
kubectl describe pod <pod-name> -n ecommerce
kubectl logs <pod-name> -n ecommerce --previous
```

### ImagePullBackOff
```bash
# Verify image exists
docker pull riunguflo/yolo-backend:v1.0.0
docker pull riunguflo/yolo-client:v1.0.0

# Check image pull policy
kubectl get pod <pod-name> -n ecommerce -o jsonpath='{.spec.containers[*].image}'
```

### ALB not provisioning
```bash
# Check AWS Load Balancer Controller
kubectl get pods -n kube-system | grep load-balancer

# Check controller logs
kubectl logs -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller --tail=100
```

### 502 Bad Gateway
- Check if pods are running: `kubectl get pods -n ecommerce`
- Check pod logs for errors
- Verify service endpoints: `kubectl get endpoints -n ecommerce`

