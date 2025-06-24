# EAIO Deployment Strategies

## Overview

This document defines the deployment architecture, infrastructure requirements, and operational strategies for the EAIO (Energy AI Optimizer) system. The deployment strategy supports both local development and enterprise-scale production environments.

## 🏗️ Deployment Architecture Principles

### 1. **Environment Consistency**
- **Infrastructure as Code**: Terraform for infrastructure provisioning
- **Configuration Management**: Consistent configurations across environments
- **Container Strategy**: Docker containerization for all services
- **Orchestration**: Kubernetes for production, Docker Compose for development

### 2. **Scalability & Reliability**
- **Horizontal Scaling**: Auto-scaling based on demand
- **Load Distribution**: Multiple availability zones and regions
- **Circuit Breakers**: Fault tolerance and graceful degradation
- **Backup & Recovery**: Automated backup and disaster recovery

---

## 🖥️ Local Development Environment

### MacBook Pro M1/M2 Deployment

#### Development Stack
```yaml
environment: local-development
hardware: MacBook Pro M1/M2 (16GB+ RAM)
os: macOS 12+ (Apple Silicon optimized)
container_runtime: Docker Desktop for Mac
orchestration: Docker Compose
```

#### Local Services Configuration
```yaml
version: '3.8'
services:
  # Core Databases
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: eaio_dev
      POSTGRES_USER: eaio_user
      POSTGRES_PASSWORD: dev_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d

  timescaledb:
    image: timescale/timescaledb:latest-pg16
    environment:
      POSTGRES_DB: eaio_timeseries
      POSTGRES_USER: timescale_user
      POSTGRES_PASSWORD: dev_password
    ports:
      - "5433:5432"
    volumes:
      - timescale_data:/var/lib/postgresql/data

  milvus-standalone:
    image: milvusdb/milvus:v2.3.0
    environment:
      ETCD_ENDPOINTS: etcd:2379
      MINIO_ADDRESS: minio:9000
    ports:
      - "19530:19530"
    depends_on:
      - etcd
      - minio

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  # Local LLM Infrastructure
  ollama:
    image: ollama/ollama:latest
    ports:
      - "11434:11434"
    volumes:
      - ollama_models:/root/.ollama
    environment:
      - OLLAMA_HOST=0.0.0.0

  # Development Services
  api-gateway:
    build: ./services/api-gateway
    ports:
      - "8080:8080"
    environment:
      - NODE_ENV=development
      - POSTGRES_URL=postgresql://eaio_user:dev_password@postgres:5432/eaio_dev
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis

  building-service:
    build: ./services/building-management
    ports:
      - "8001:8001"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://eaio_user:dev_password@postgres:5432/eaio_dev
    depends_on:
      - postgres

volumes:
  postgres_data:
  timescale_data:
  redis_data:
  ollama_models:
```

#### Local Development Commands
```bash
# Start development environment
docker-compose up -d

# Initialize databases with schema
docker-compose exec postgres psql -U eaio_user -d eaio_dev -f /docker-entrypoint-initdb.d/schema.sql

# Load BDG2 sample data
docker-compose exec timescaledb psql -U timescale_user -d eaio_timeseries -f /docker-entrypoint-initdb.d/bdg2_sample.sql

# Start local LLM models
docker-compose exec ollama ollama pull llama3.2:3b
docker-compose exec ollama ollama pull qwen2.5:7b

# Run frontend development server
cd frontend && npm run dev

# Run Streamlit analytics
cd analytics && streamlit run app.py --server.port 8501
```

### Development Workflow
```mermaid
graph TD
    A[Code Changes] --> B[Local Testing]
    B --> C[Unit Tests]
    C --> D[Integration Tests]
    D --> E[Docker Build]
    E --> F[Local Deploy]
    F --> G[E2E Testing]
    G --> H[Git Commit]
    H --> I[CI/CD Pipeline]
```

---

## ☁️ Cloud Production Environment

### AWS Enterprise Deployment

#### Infrastructure Overview
```yaml
cloud_provider: AWS
regions: [us-east-1, us-west-2]
availability_zones: 3 per region
container_orchestration: Amazon EKS
service_mesh: Istio
monitoring: Prometheus + Grafana
logging: ELK Stack
secrets_management: AWS Secrets Manager
```

#### EKS Cluster Configuration
```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: eaio-production
  region: us-east-1
  version: "1.27"

managedNodeGroups:
  - name: worker-nodes
    instanceType: c6i.2xlarge
    minSize: 3
    maxSize: 20
    desiredCapacity: 6
    volumeSize: 100
    ssh:
      allow: false
    iam:
      withAddonPolicies:
        ebs: true
        fsx: true
        efs: true

  - name: ai-nodes
    instanceType: c6i.4xlarge
    minSize: 2
    maxSize: 10
    desiredCapacity: 4
    volumeSize: 200
    labels:
      workload: ai-ml
    taints:
      ai-workload: "true:NoSchedule"

addons:
  - name: vpc-cni
  - name: coredns
  - name: kube-proxy
  - name: aws-ebs-csi-driver
```

#### Database Deployment
```yaml
# Amazon RDS for PostgreSQL
postgres_primary:
  engine: postgres
  version: "16.1"
  instance_class: db.r6g.2xlarge
  allocated_storage: 1000
  storage_encrypted: true
  backup_retention: 30
  multi_az: true
  performance_insights: true

# Amazon RDS for TimescaleDB
timescaledb:
  engine: postgres
  version: "16.1"
  instance_class: db.r6g.4xlarge
  allocated_storage: 5000
  storage_encrypted: true
  parameter_group: timescaledb-optimized

# Amazon MemoryDB for Redis
redis_cluster:
  node_type: db.r6g.large
  num_shards: 3
  num_replicas: 2
  at_rest_encryption: true
  transit_encryption: true

# Milvus on EKS
milvus_cluster:
  replicas: 3
  storage_class: gp3
  storage_size: 2Ti
  memory_request: 16Gi
  memory_limit: 32Gi
```

### Production Services Deployment

#### Kubernetes Manifests
```yaml
# Building Management Service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: building-service
  namespace: eaio-production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: building-service
  template:
    metadata:
      labels:
        app: building-service
    spec:
      containers:
      - name: building-service
        image: eaio/building-service:v1.1.0
        ports:
        - containerPort: 8001
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-secrets
              key: postgres-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: cache-secrets
              key: redis-url
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8001
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8001
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: building-service
  namespace: eaio-production
spec:
  selector:
    app: building-service
  ports:
  - port: 8001
    targetPort: 8001
  type: ClusterIP

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: building-service-hpa
  namespace: eaio-production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: building-service
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

#### AI/ML Services Deployment
```yaml
# LLM Router Service with GPU support
apiVersion: apps/v1
kind: Deployment
metadata:
  name: llm-router-service
  namespace: eaio-ai
spec:
  replicas: 2
  selector:
    matchLabels:
      app: llm-router-service
  template:
    metadata:
      labels:
        app: llm-router-service
    spec:
      nodeSelector:
        workload: ai-ml
      tolerations:
      - key: ai-workload
        operator: Equal
        value: "true"
        effect: NoSchedule
      containers:
      - name: llm-router
        image: eaio/llm-router:v1.1.0
        ports:
        - containerPort: 8008
        env:
        - name: OLLAMA_HOST
          value: "ollama-service:11434"
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: llm-secrets
              key: openai-api-key
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
            nvidia.com/gpu: 1

# Ollama Deployment for Local Models
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ollama
  namespace: eaio-ai
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ollama
  template:
    metadata:
      labels:
        app: ollama
    spec:
      nodeSelector:
        workload: ai-ml
      containers:
      - name: ollama
        image: ollama/ollama:latest
        ports:
        - containerPort: 11434
        env:
        - name: OLLAMA_HOST
          value: "0.0.0.0"
        volumeMounts:
        - name: model-storage
          mountPath: /root/.ollama
        resources:
          requests:
            memory: "8Gi"
            cpu: "2000m"
          limits:
            memory: "16Gi"
            cpu: "4000m"
            nvidia.com/gpu: 1
      volumes:
      - name: model-storage
        persistentVolumeClaim:
          claimName: ollama-models-pvc
```

---

## 🌍 Multi-Environment Strategy

### Environment Progression
```mermaid
graph LR
    A[Local Dev] --> B[Development]
    B --> C[Staging]
    C --> D[Production]
    D --> E[DR Site]
    
    F[Feature Branch] --> A
    G[Main Branch] --> B
    H[Release Branch] --> C
    I[Tagged Release] --> D
```

### Environment Configurations

#### Development Environment
```yaml
environment: development
namespace: eaio-dev
replicas: 1
resources:
  requests: minimal
  limits: low
database: shared-dev-instance
monitoring: basic
ssl: self-signed
domain: dev.eaio.internal
```

#### Staging Environment
```yaml
environment: staging
namespace: eaio-staging
replicas: 2
resources:
  requests: production-like
  limits: production-like
database: dedicated-staging
monitoring: full-stack
ssl: letsencrypt
domain: staging.eaio.com
data: production-like-sample
```

#### Production Environment
```yaml
environment: production
namespace: eaio-prod
replicas: 3+
resources:
  requests: optimized
  limits: high-performance
database: multi-az-cluster
monitoring: enterprise-grade
ssl: enterprise-certificate
domain: app.eaio.com
data: real-production-data
backup: automated-daily
```

---

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow
```yaml
name: EAIO CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  release:
    types: [published]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: [building-service, energy-service, llm-router]
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: services/${{ matrix.service }}/package-lock.json
    
    - name: Install dependencies
      run: |
        cd services/${{ matrix.service }}
        npm ci
    
    - name: Run tests
      run: |
        cd services/${{ matrix.service }}
        npm run test:coverage
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        file: services/${{ matrix.service }}/coverage/lcov.info

  build:
    needs: test
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: Login to Amazon ECR
      uses: aws-actions/amazon-ecr-login@v2
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ secrets.ECR_REGISTRY }}/eaio
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=semver,pattern={{version}}
          type=sha
    
    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
    - name: Deploy to Staging
      run: |
        aws eks update-kubeconfig --name eaio-staging
        kubectl set image deployment/building-service building-service=${{ needs.build.outputs.image-tag }}
        kubectl rollout status deployment/building-service

  deploy-production:
    if: github.event_name == 'release'
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
    - name: Deploy to Production
      run: |
        aws eks update-kubeconfig --name eaio-production
        kubectl set image deployment/building-service building-service=${{ needs.build.outputs.image-tag }}
        kubectl rollout status deployment/building-service
```

### Deployment Automation
```bash
#!/bin/bash
# deploy.sh - Automated deployment script

set -e

ENVIRONMENT=$1
SERVICE=$2
VERSION=$3

echo "Deploying $SERVICE v$VERSION to $ENVIRONMENT"

# Update Kubernetes configuration
kubectl config use-context eaio-$ENVIRONMENT

# Apply configuration changes
kubectl apply -f k8s/$ENVIRONMENT/

# Update image version
kubectl set image deployment/$SERVICE $SERVICE=eaio/$SERVICE:$VERSION

# Wait for rollout to complete
kubectl rollout status deployment/$SERVICE --timeout=300s

# Run health checks
kubectl exec deployment/$SERVICE -- curl -f http://localhost:8001/health

echo "Deployment completed successfully"
```

---

## 📊 Infrastructure Monitoring

### Monitoring Stack
```yaml
# Prometheus Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true

# Grafana Dashboard for EAIO
apiVersion: v1
kind: ConfigMap
metadata:
  name: eaio-dashboard
data:
  dashboard.json: |
    {
      "dashboard": {
        "title": "EAIO System Overview",
        "panels": [
          {
            "title": "Service Response Times",
            "type": "graph",
            "targets": [
              {
                "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
              }
            ]
          },
          {
            "title": "Database Performance",
            "type": "graph",
            "targets": [
              {
                "expr": "rate(postgresql_queries_total[5m])"
              }
            ]
          },
          {
            "title": "LLM Processing Time",
            "type": "graph", 
            "targets": [
              {
                "expr": "histogram_quantile(0.95, rate(llm_request_duration_seconds_bucket[5m]))"
              }
            ]
          }
        ]
      }
    }
```

### Health Check Configuration
```typescript
// Health check endpoints for all services
interface HealthCheck {
  status: 'healthy' | 'unhealthy' | 'degraded'
  timestamp: string
  version: string
  uptime: number
  dependencies: {
    database: 'connected' | 'disconnected'
    cache: 'connected' | 'disconnected'
    external_apis: 'available' | 'unavailable'
  }
  metrics: {
    memory_usage: number
    cpu_usage: number
    response_time: number
    error_rate: number
  }
}
```

---

## 🔒 Security & Compliance

### Security Measures
```yaml
# Network Policies
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: eaio-network-policy
spec:
  podSelector:
    matchLabels:
      app: eaio
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: eaio-production
    ports:
    - protocol: TCP
      port: 8080

# Pod Security Policy
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: eaio-psp
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
```

### Compliance Controls
- **Data Encryption**: AES-256 at rest, TLS 1.3 in transit
- **Access Control**: RBAC with principle of least privilege
- **Audit Logging**: Complete audit trail for all operations
- **Backup & Recovery**: Automated backups with tested recovery procedures
- **Incident Response**: 24/7 monitoring with automated alerting

---

## 📈 Performance & Scaling

### Auto-scaling Configuration
```yaml
# Cluster Autoscaler
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-autoscaler-status
  namespace: kube-system
data:
  nodes.max: "100"
  nodes.min: "10"
  scale-down-delay-after-add: "10m"
  scale-down-unneeded-time: "10m"

# Vertical Pod Autoscaler
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: building-service-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: building-service
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: building-service
      maxAllowed:
        cpu: "2"
        memory: 4Gi
      minAllowed:
        cpu: 100m
        memory: 256Mi
```

### Performance Targets
| Environment | Latency | Throughput | Availability | RPO | RTO |
|-------------|---------|------------|--------------|-----|-----|
| **Development** | <2s | 100 RPS | 95% | 24h | 4h |
| **Staging** | <1s | 500 RPS | 99% | 1h | 1h |
| **Production** | <500ms | 1000+ RPS | 99.9% | 15min | 30min |

This comprehensive deployment strategy ensures reliable, scalable, and secure operation of the EAIO system across all environments from local development to enterprise production.
