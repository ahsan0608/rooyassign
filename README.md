## Part 1 - Kubernetes Workloads Challenge

### Overview
Implementation of Kubernetes manifests for a secure and well-structured application deployment.

### Components

1. **Backend Deployment**
   - Immutable deployment with read-only filesystem
   - Writable /tmp directory
   - Liveness and Readiness probes
   - Security configurations

2. **Database Services**
   - db1 deployment with nginx on port 6379
   - db2 deployment with nginx on port 5432
   - Custom nginx configurations
   - Secret management for db2

3. **Network Policies**
   - Restricted pod-to-pod communication
   - Secure access control between services

### Directory Structure
```
kubernetes/
├── manifests/
│   ├── backend.yaml      # Backend deployment
│   ├── db.yaml          # Database deployments and services
│   ├── network-policy.yaml  # Network policies
│   └── secrets.yaml     # Secret configurations
├── helm/
│   ├── postgres-values.yaml    # Postgres Helm values
│   └── prometheus-values.yaml  # Prometheus stack values
```

### Verification Steps

1. **Deploy the Manifests**
```bash
# Create namespace
kubectl apply -f kubernetes/manifests/backend.yaml

# Deploy databases
kubectl apply -f kubernetes/manifests/db.yaml

# Apply network policies
kubectl apply -f kubernetes/manifests/network-policy.yaml

# Create secrets
kubectl apply -f kubernetes/manifests/secrets.yaml
```

2. **Verify Deployments**
```bash
# Check pod status
kubectl get pods -n project-plato

# Verify services
kubectl get services -n project-plato

# Check network policies
kubectl get networkpolicies -n project-plato
```

3. **Test Connectivity**
```bash
# Test backend readiness
kubectl exec -n project-plato <backend-pod> -- nc -zv db1 6379

# Verify db2 secrets
kubectl exec -n project-plato <db2-pod> -- env | grep DB_
```

### Bonus Tasks Implementation

#### 1. Deploy Postgres with Helm
```bash
# Add Bitnami Helm repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Install Postgres with metrics enabled
helm install postgres bitnami/postgresql \
  -f kubernetes/helm/postgres-values.yaml \
  -n project-plato

# Verify Postgres deployment
kubectl get pods -n project-plato -l app.kubernetes.io/name=postgresql
kubectl get svc -n project-plato -l app.kubernetes.io/name=postgresql
```

Key features enabled:
- PostgreSQL metrics exporter
- ServiceMonitor for Prometheus integration
- Resource limits and requests

#### 2. Deploy Prometheus Stack
```bash
# Add Prometheus community Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus stack with Postgres monitoring
helm install prometheus prometheus-community/kube-prometheus-stack \
  -f kubernetes/helm/prometheus-values.yaml \
  -n project-plato

# Verify Prometheus stack deployment
kubectl get pods -n project-plato -l app=prometheus
kubectl get pods -n project-plato -l app=grafana
```

Components deployed:
- Prometheus Operator
- Prometheus Server
- Grafana with PostgreSQL dashboard
- AlertManager

#### 3. Access and Verify Monitoring

1. **Access Prometheus UI**:
```bash
# Port forward Prometheus service
kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090 -n project-plato
```
- Visit: http://localhost:9090
- Verify target status: Status → Targets (check if postgres-metrics is up)

2. **Access Grafana Dashboard**:
```bash
# Get Grafana admin password
kubectl get secret prometheus-grafana -n project-plato -o jsonpath="{.data.admin-password}" | base64 -d

# Port forward Grafana service
kubectl port-forward svc/prometheus-grafana 3000:80 -n project-plato
```
- Visit: http://localhost:3000
- Login with:
  - Username: admin
  - Password: [from above command]
- Navigate to Dashboards → PostgreSQL Metrics

3. **Verify Postgres Metrics**
Common PromQL queries to verify:
```promql
# Check if Postgres exporter is running
pg_up

# Database size
pg_database_size_bytes

# Transaction metrics
pg_stat_database_tup_fetched
pg_stat_database_tup_inserted
pg_stat_database_tup_updated
pg_stat_database_tup_deleted

# Connection metrics
pg_stat_database_numbackends
```

4. **Test Alerting**:
```bash
# View configured alerts
kubectl port-forward svc/prometheus-kube-prometheus-alertmanager 9093:9093 -n project-plato
```
- Visit: http://localhost:9093
- Check configured alerts under "Rules"

#### Configuration Files

1. **postgres-values.yaml**:
- Metrics exporter configuration
- ServiceMonitor setup
- Resource limits
- Authentication details

2. **prometheus-values.yaml**:
- Prometheus Operator settings
- Grafana dashboard configuration
- AlertManager rules
- PostgreSQL monitoring integration

## Part 2 - Production Infrastructure Design

### Key Components

1. **Developer Self-Management**
   - GitOps workflow with ArgoCD
   - CI/CD pipeline using GitHub Actions
   - Container registry integration
   - RBAC for access control

2. **Deployment Technologies**
   - Kubernetes (EKS/GKE/AKS)
   - Istio Service Mesh
   - Ingress Controller (NGINX/Traefik)
   - Cert-Manager for TLS
   - External DNS

3. **Security & Observability**
   - HashiCorp Vault
   - Network Policies
   - Prometheus & Grafana
   - ELK Stack for logging

### Deployment Workflow

1. Code Commit → GitHub/GitLab
2. CI/CD Pipeline:
   - Automated builds
   - Security scanning
   - Container image creation
3. GitOps Deployment:
   - ArgoCD synchronization
   - Kubernetes deployment
   - Service mesh configuration
   ![GitOps Flow](https://github.com/user-attachments/assets/90b5f065-3aae-4a78-b4f3-a556105e70e8)
4. Public Access:
   - Ingress routing
   - TLS termination
   - DNS management
5. Monitoring:
   - Metrics collection
   - Log aggregation
   - Performance monitoring
   ![Production Deployment Architecture](https://github.com/user-attachments/assets/987f8bae-2c98-44e3-aefb-79e737114eae)

### Security Measures

1. **Access Control**
   - RBAC implementation
   - Service mesh mTLS
   - Network policies

2. **Secret Management**
   - HashiCorp Vault integration
   - Encrypted secrets
   - Secure key rotation

3. **Network Security**
   - Ingress protection
   - Pod-to-pod isolation
   - External traffic control

### Observability Stack

1. **Monitoring**
   - Real-time metrics
   - Custom dashboards
   - Alert management

2. **Logging**
   - Centralized logging
   - Log retention
   - Audit trails

3. **Tracing**
   - Request tracking
   - Performance analysis
   - Dependency mapping

### Implementation Phases

1. Core Infrastructure Setup
2. Security Layer Implementation
3. Developer Platform Deployment
4. Observability Stack Configuration
5. High Availability Setup
6. Progressive Delivery Implementation
7. Documentation & Training

### Bonus Task: Partial Implementation

#### 1. Convert Task 1 Manifests to GitOps Structure
```
infrastructure/
├── base/                    # Base configurations
│   ├── backend/
│   ├── databases/
│   └── monitoring/
├── overlays/               # Environment-specific configs
│   ├── development/
│   ├── staging/
│   └── production/
└── platform/              # Shared infrastructure
    ├── ingress-nginx/
    ├── cert-manager/
    └── monitoring/
```

#### 2. Production Components
Essential components to implement:

1. **Access & Security**
```bash
# Install Ingress Controller
helm install ingress-nginx ingress-nginx/ingress-nginx

# Install Cert-Manager
helm install cert-manager jetstack/cert-manager --set installCRDs=true

# Configure basic auth
kubectl create secret generic basic-auth --from-file=auth
```

2. **Monitoring Extension**
```bash
# Install Grafana Loki for logs
helm install loki grafana/loki-stack

# Add service monitors for existing components
kubectl apply -f infrastructure/platform/monitoring/
```

3. **GitOps Setup**
```bash
# Install ArgoCD
helm install argo-cd argo/argo-cd

# Configure application manifests
kubectl apply -f infrastructure/base/argocd/applications/
```

#### 3. Quick Start
```bash
# 1. Install core infrastructure
./scripts/install-core.sh

# 2. Deploy Task 1 components through ArgoCD
kubectl apply -f infrastructure/base/argocd/apps/task1.yaml

# 3. Access the application
kubectl get ingress
```
