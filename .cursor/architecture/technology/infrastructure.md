# EAIO Infrastructure Requirements

## Overview

This document defines infrastructure requirements for the EAIO system across local development and cloud production environments.

## 💻 Local Development Infrastructure

### MacBook Pro M1/M2 Requirements

#### Hardware Specifications
```yaml
processor: Apple M1/M2 (8-core CPU minimum)
memory: 16GB RAM (24GB+ recommended)
storage: 512GB SSD (1TB+ recommended)
os: macOS 12+ with Apple Silicon support
```

#### Resource Allocation
```yaml
docker_resources:
  memory: 8GB
  cpu: 6 cores
  storage: 100GB

services:
  postgresql: { memory: 512MB, cpu: 0.5, storage: 10GB }
  timescaledb: { memory: 1GB, cpu: 1.0, storage: 20GB }
  milvus: { memory: 2GB, cpu: 1.0, storage: 10GB }
  redis: { memory: 256MB, cpu: 0.25, storage: 1GB }
  ollama: { memory: 8GB, cpu: 2.0, storage: 20GB }
```

#### Port Configuration
```yaml
services:
  postgresql: 5432
  timescaledb: 5433
  milvus: 19530
  redis: 6379
  ollama: 11434
  api_gateway: 8080
  frontend: 3000
  streamlit: 8501
```

---

## ☁️ Cloud Production Infrastructure

### AWS Architecture

#### Core Infrastructure
```yaml
# VPC Configuration
vpc:
  cidr: 10.0.0.0/16
  availability_zones: 3
  public_subnets: [10.0.1.0/24, 10.0.2.0/24, 10.0.3.0/24]
  private_subnets: [10.0.10.0/24, 10.0.20.0/24, 10.0.30.0/24]

# EKS Cluster
eks:
  name: eaio-production
  version: "1.28"
  worker_nodes:
    instance_type: c6i.2xlarge
    min: 3, max: 20, desired: 6
  ai_nodes:
    instance_type: c6i.4xlarge
    min: 2, max: 10, desired: 4
```

#### Database Infrastructure
```yaml
# PostgreSQL Primary
rds_postgresql:
  engine: postgres 16.1
  instance: db.r6g.2xlarge
  storage: 1TB (gp3, encrypted)
  multi_az: true
  backup_retention: 30d

# TimescaleDB
rds_timescaledb:
  engine: postgres 16.1
  instance: db.r6g.4xlarge
  storage: 5TB (gp3, encrypted)
  read_replicas: 2

# Redis Cluster
elasticache:
  engine: redis 7.0
  node_type: cache.r6g.large
  replication_groups: 3
  encryption: enabled
```

#### Storage & Networking
```yaml
# Milvus Vector Database
milvus:
  deployment: cluster
  replicas: 3
  storage: 2TB EFS
  performance: generalPurpose

# Load Balancing
alb:
  scheme: internet-facing
  listeners: [443/HTTPS, 80/HTTP->443]
  targets: [api-gateway:8080, frontend:3000]

# CDN
cloudfront:
  static_cache_ttl: 86400
  api_cache_ttl: 0
  price_class: PriceClass_100
```

---

## 📊 Capacity Planning

### Production Resource Allocation
```yaml
service_resources:
  api_gateway:
    cpu: 500m, memory: 1Gi, replicas: 3
    autoscaling: { min: 3, max: 10, cpu: 70% }
    
  building_service:
    cpu: 500m, memory: 512Mi, replicas: 3
    autoscaling: { min: 3, max: 15, cpu: 70% }
    
  energy_service:
    cpu: 1000m, memory: 2Gi, replicas: 5
    autoscaling: { min: 5, max: 20, cpu: 60% }
    
  llm_router:
    cpu: 1000m, memory: 2Gi, replicas: 2
    autoscaling: { min: 2, max: 8, cpu: 60% }
    
  ollama_local:
    cpu: 4000m, memory: 16Gi, replicas: 2
    autoscaling: { min: 2, max: 4, memory: 85% }
```

### Performance Targets
```yaml
latency_sla:
  api_response: 95th < 200ms
  database_query: 95th < 100ms
  cache_lookup: 99th < 5ms
  agent_processing: 95th < 5s

throughput_sla:
  api_requests: 1000 RPS
  database_writes: 10000 TPS
  real_time_streams: 50000 events/sec

availability_sla:
  overall_system: 99.9%
  database: 99.95%
  cache: 99.9%
  ml_processing: 99.5%
```

---

## 🔐 Security Infrastructure

### Network Security
```yaml
security_groups:
  eks_cluster:
    ingress: [443/HTTPS from VPC]
    egress: [all]
    
  databases:
    ingress: [5432/TCP from EKS]
    egress: []

waf_protection:
  rules: [CommonRuleSet, KnownBadInputs, SQLiRuleSet]
  rate_limiting: 2000_requests_per_5min
```

### Encryption
```yaml
at_rest:
  databases: AES-256 (KMS)
  storage: AES-256 (SSE-S3)
  
in_transit:
  api: TLS 1.3
  database: SSL/TLS
  inter_service: mTLS (Istio)
```

---

## 📈 Monitoring Infrastructure

### Observability Stack
```yaml
prometheus:
  retention: 30d
  storage: 100GB
  replicas: 2

grafana:
  dashboards: [system, database, api, ml]
  storage: 20GB

elk_stack:
  elasticsearch: 3 nodes, 500GB each
  logstash: 2 replicas
  kibana: 2 replicas
  
jaeger:
  tracing: distributed
  sampling: 10%
  retention: 7d
```

---

## 💰 Cost Management

### Monthly Cost Estimates
```yaml
aws_costs:
  compute: $3000
  database: $2500
  storage: $800
  networking: $500
  monitoring: $400
  total_monthly: $7200

optimization:
  spot_instances: 30% (batch workloads)
  reserved_instances: 50% (1-year term)
  intelligent_tiering: S3 storage
  rightsizing: continuous monitoring
```

---

## 🔄 Backup & Disaster Recovery

### Backup Strategy
```yaml
databases:
  postgresql: daily, 30d retention, cross-region
  timescaledb: 6h frequency, 7d retention
  
application:
  snapshots: weekly, 12 weeks retention
  
recovery_objectives:
  rpo: 15 minutes
  rto: 30 minutes
```

### Multi-Region DR
```yaml
setup:
  primary: us-east-1
  backup: us-west-2
  strategy: active-passive
  
failover:
  databases: automated read replica promotion
  applications: terraform deployment
  dns: route53 health checks
  downtime: <15 minutes
```

This infrastructure provides enterprise-grade scalability, security, and reliability for the EAIO system. 