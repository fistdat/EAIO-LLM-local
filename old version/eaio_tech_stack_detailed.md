# EAIO Tech Stack - Kiến Trúc Kỹ Thuật Chi Tiết
**Updated for MCP Integration & Local LLM Deployment**

## 🏗️ Tổng Quan Kiến Trúc

Hệ thống EAIO được thiết kế theo **4-Tier MCP-Integrated Multi-Agent Architecture** với local LLM deployment cho MacBook Pro M1, đảm bảo:
- **Real-time performance** (<2s response time)
- **Data privacy** (local LLM processing)
- **Standardized tool integration** (MCP protocol)
- **Hardware optimization** (M1 16GB RAM constraints)
- **Scalability** (support >1,000 buildings locally)
- **Reliability** (99.9% uptime)

---

## 🧠 Layer 1: LLM Infrastructure (Core Engine)

### Local LLM Deployment Stack
```yaml
Primary LLM Models:
  - DeepSeek-V3-70B: Coordinator & Strategic Agents  
  - DeepSeek-V3-33B: Complex reasoning tasks
  - Llama3-8B: Operational agents & real-time tasks
  - Llama3-70B: Backup for critical operations

Inference Engine:
  - vLLM v0.6.0: 2.7x throughput improvement
  - Ollama: Local model serving
  - LiteLLM: Multi-model proxy
  - OpenAI-compatible API wrapper

Hardware Configuration:
  - Coordinator Tier: 2x NVIDIA H100 (160GB VRAM)
  - Worker Tier: 8x NVIDIA RTX 4090 (192GB VRAM) 
  - CPU: 2x AMD EPYC 7763 (128 cores total)
  - RAM: 1TB DDR4 ECC
  - Storage: 20TB NVMe SSD RAID 10
```

### Model Performance Specifications
| Model | Parameters | VRAM | Tokens/sec | Use Case |
|-------|------------|------|------------|----------|
| DeepSeek-V3-70B | 70B | 35GB | 25 | Strategic Planning |
| DeepSeek-V3-33B | 33B | 17GB | 45 | Customer Service |
| Llama3-8B | 8B | 8GB | 76 | Real-time Operations |

---

## 🔗 Layer 2: MCP Integration Layer

### Model Context Protocol Implementation
```typescript
MCP Architecture:
  - Protocol: JSON-RPC 2.0 over HTTP/SSE
  - Transport: stdio/HTTP hybrid for reliability
  - Security: mTLS + JWT authentication
  - Load Balancing: Nginx with least-connections

MCP Server Configuration:
  - Energy Data Server: BACnet/Modbus integration
  - Weather API Server: OpenWeatherMap + NOAA
  - Market Data Server: Real-time energy pricing
  - IoT Sensor Server: MQTT broker integration
  - Analytics Server: Custom ML model serving
```

### MCP Tools & Resources
```python
# Core MCP Tools
energy_consumption_tool = MCPTool(
    name="get_energy_consumption",
    description="Retrieve building energy usage data",
    parameters={"building_id", "time_range", "meter_type"}
)

weather_forecast_tool = MCPTool(
    name="get_weather_forecast", 
    description="Get weather predictions for optimization",
    parameters={"location", "forecast_hours", "parameters"}
)

optimization_tool = MCPTool(
    name="optimize_energy_usage",
    description="Multi-objective energy optimization",
    parameters={"constraints", "objectives", "time_horizon"}
)
```

---

## 🤖 Layer 3: Multi-Agent Framework

### Agent Framework Stack
```yaml
Primary Framework: LangGraph v0.2.x
  - Pros: Native MCP support, state management
  - Cons: Learning curve, newer ecosystem
  
Alternative Framework: AutoGen v0.2.x
  - Pros: Mature ecosystem, conversation patterns
  - Cons: No native MCP, requires custom integration

Agent Communication:
  - Message Broker: Apache Kafka 3.7
  - Topic Structure: agent-{tier}-{function}
  - Protocol: Apache Avro serialization
  - Coordination: Consensus via Raft algorithm
```

### Agent Configuration Template
```python
# Tier 1 Agent Example - Data Manager
data_manager = LlmAgent(
    model="llama3-8b",
    name="data_manager",
    description="IoT data processing and quality assurance",
    instruction="""
    You are the Data Manager Agent responsible for:
    1. Processing IoT sensor data streams
    2. Ensuring data quality and completeness  
    3. Real-time anomaly detection in data feeds
    4. Coordinating with MCP servers for data access
    """,
    tools=[
        iot_data_collector,
        data_quality_checker, 
        real_time_processor
    ],
    generate_content_config=GenerateContentConfig(
        temperature=0.1,  # Low for deterministic data processing
        max_output_tokens=500
    )
)
```

---

## 💾 Layer 4: Data Infrastructure

### Database Architecture
```yaml
Time-Series Database:
  - Primary: InfluxDB v2.7 (energy metrics)
  - Retention: 1 year detailed, 5 years aggregated
  - Partitioning: By building_id and time
  - Replication: 3x for high availability

Vector Database:
  - Primary: ChromaDB v0.4 (embeddings)
  - Use: Document search, semantic similarity
  - Models: all-MiniLM-L6-v2 embeddings
  - Index: HNSW for fast similarity search

Graph Database:
  - Primary: Neo4j v5.x (relationships)
  - Use: Building topology, energy flow
  - Queries: Cypher for path optimization
  - Clustering: Core cluster with read replicas

Relational Database:
  - Primary: PostgreSQL v16 (metadata)
  - Extensions: TimescaleDB, PostGIS
  - Partitioning: By date and building_id
  - Backup: Point-in-time recovery

Cache Layer:
  - Redis Cluster v7.x (session, real-time data)
  - Configuration: 6 nodes (3 master, 3 replica)
  - Persistence: RDB snapshots + AOF
  - Memory: 128GB total
```

### Data Processing Pipeline
```python
# Real-time Stream Processing
@streaming_processor
def process_iot_data(sensor_data):
    """Process IoT sensor data in real-time"""
    
    # Data validation
    validated_data = validate_sensor_reading(sensor_data)
    
    # Store in time-series DB
    influx_client.write_points([{
        'measurement': 'energy_consumption',
        'tags': {
            'building_id': sensor_data.building_id,
            'meter_type': sensor_data.meter_type
        },
        'fields': {
            'value': validated_data.value,
            'quality': validated_data.quality_score
        },
        'time': sensor_data.timestamp
    }])
    
    # Trigger anomaly detection
    if anomaly_detector.detect(validated_data):
        publish_alert(validated_data)
    
    # Update real-time cache
    redis_client.setex(
        f"latest:{sensor_data.building_id}:{sensor_data.meter_type}",
        300,  # 5 minute TTL
        validated_data.to_json()
    )
```

---

## 🌐 Layer 5: Integration & Communication

### Building Management System Integration
```yaml
Protocol Support:
  - BACnet/IP: Primary building automation protocol
  - Modbus TCP: Industrial equipment communication  
  - MQTT: IoT sensor data collection
  - OPC UA: Modern industrial communication
  - REST APIs: Cloud service integration

Network Architecture:
  - DMZ: Public-facing web interface
  - Management: Agent communication network
  - OT Network: Building system integration
  - Firewall: Segmented security zones
```

### External API Integration
```python
# Weather API Integration
@mcp_tool
async def get_weather_forecast(location: str, hours: int = 24):
    """Get weather forecast for energy optimization"""
    
    async with aiohttp.ClientSession() as session:
        # NOAA API for US locations
        if is_us_location(location):
            url = f"https://api.weather.gov/gridpoints/{location}/forecast"
            async with session.get(url) as response:
                data = await response.json()
                
        # OpenWeatherMap for international
        else:
            url = f"https://api.openweathermap.org/data/2.5/forecast"
            params = {
                'q': location,
                'appid': os.getenv('OPENWEATHER_API_KEY'),
                'units': 'metric'
            }
            async with session.get(url, params=params) as response:
                data = await response.json()
    
    return parse_weather_data(data)

# Energy Market API Integration  
@mcp_tool
async def get_energy_prices(market: str, hours: int = 24):
    """Get real-time energy market pricing"""
    
    market_apis = {
        'CAISO': 'http://oasis.caiso.com/oasisapi/SingleZip',
        'PJM': 'https://api.pjm.com/api/v1/',
        'ERCOT': 'http://mis.ercot.com/misapp/GetReports.do'
    }
    
    # Implementation varies by market operator
    return await fetch_market_data(market_apis[market])
```

---

## 🔧 Layer 6: DevOps & Infrastructure

### Containerization & Orchestration
```yaml
Container Runtime:
  - Docker v24.x with BuildKit
  - Base Images: Ubuntu 22.04 LTS minimal
  - Multi-stage builds for optimization
  - Security scanning with Trivy

Orchestration:
  - Kubernetes v1.29
  - Cluster: 5 nodes (3 master, 2 worker)
  - CNI: Calico for network policies
  - Storage: Longhorn distributed storage
  - Ingress: Nginx Ingress Controller

Deployment Strategy:
  - Blue-Green deployment for agents
  - Rolling updates for infrastructure
  - Canary releases for new models
  - Automatic rollback on failure
```

### Monitoring & Observability
```yaml
Metrics Collection:
  - Prometheus v2.x: Time-series metrics
  - Grafana v10.x: Visualization dashboards
  - AlertManager: Intelligent alerting
  - Node Exporter: Infrastructure metrics

Distributed Tracing:
  - Jaeger v1.x: Request tracing
  - OpenTelemetry: Instrumentation standard
  - Zipkin: Legacy trace compatibility
  - Custom spans: Agent conversation tracking

Logging:
  - ELK Stack: Elasticsearch + Logstash + Kibana
  - FluentBit: Log collection and forwarding
  - Log levels: ERROR, WARN, INFO, DEBUG
  - Retention: 30 days operational, 1 year compliance

Application Performance:
  - Custom metrics: Agent response times
  - LLM performance: Tokens/second, latency
  - Energy optimization: Savings percentage
  - User experience: Dashboard load times
```

---

## 🔒 Layer 7: Security & Compliance

### Security Architecture
```yaml
Network Security:
  - Zero-Trust Architecture
  - mTLS for all inter-service communication
  - Network segmentation with microsegmentation
  - WAF (Web Application Firewall) protection

Identity & Access Management:
  - OAuth 2.0 + OIDC for authentication
  - RBAC (Role-Based Access Control)
  - MFA (Multi-Factor Authentication) required
  - SAML integration for enterprise SSO

Data Protection:
  - Encryption at rest: AES-256
  - Encryption in transit: TLS 1.3
  - Key management: HashiCorp Vault
  - Data classification: Public, Internal, Confidential

Compliance Framework:
  - NERC CIP: Critical Infrastructure Protection
  - ISO 27001: Information Security Management
  - SOC 2 Type II: Service Organization Controls
  - GDPR: Data protection for EU operations
```

### Security Monitoring
```python
# Security Event Detection
@security_monitor
def detect_anomalous_behavior(agent_activity):
    """Monitor agent behavior for security threats"""
    
    # Unusual authentication patterns
    if detect_brute_force(agent_activity.auth_attempts):
        trigger_account_lockout(agent_activity.user_id)
    
    # Suspicious data access patterns
    if detect_data_exfiltration(agent_activity.data_requests):
        quarantine_session(agent_activity.session_id)
    
    # Abnormal agent communication
    if detect_command_injection(agent_activity.messages):
        isolate_agent(agent_activity.agent_id)
    
    # Performance anomalies indicating compromise
    if detect_resource_abuse(agent_activity.resource_usage):
        scale_down_agent(agent_activity.agent_id)
```

---

## 📊 Layer 8: Analytics & Reporting

### Business Intelligence Stack
```yaml
Data Warehouse:
  - Primary: PostgreSQL with Timescale
  - ETL: Apache Airflow for data pipelines
  - Real-time: Apache Kafka + ksqlDB
  - Analytics: Apache Superset dashboards

Machine Learning Pipeline:
  - Training: MLflow + Kubeflow
  - Feature Store: Feast
  - Model Registry: MLflow Model Registry
  - Serving: Seldon Core on Kubernetes

Custom Analytics:
  - Energy savings calculations
  - Carbon footprint tracking
  - Predictive maintenance schedules
  - ROI analysis and forecasting
```

---

## 🚀 Layer 9: API & Frontend

### API Architecture
```yaml
API Gateway:
  - Kong or Ambassador for API management
  - Rate limiting: 1000 req/min per user
  - Authentication: JWT tokens
  - Documentation: OpenAPI 3.0 specs

GraphQL Layer:
  - Apollo Server for unified data access
  - Real-time subscriptions for live data
  - DataLoader for efficient batching
  - Schema federation for microservices

REST APIs:
  - FastAPI for high-performance endpoints
  - Pydantic for data validation
  - Async/await for non-blocking operations
  - Auto-generated documentation
```

### Frontend Technology Stack
```yaml
Web Application:
  - Framework: React 18 with TypeScript
  - State Management: Redux Toolkit + RTK Query
  - UI Components: Material-UI (MUI) v5
  - Charts: Recharts + D3.js for custom visualizations
  - Real-time: WebSocket + SSE for live updates

Dashboard Features:
  - Real-time energy consumption monitoring
  - Interactive building floor plans
  - Predictive analytics visualizations
  - Alert management interface
  - Agent conversation history
  - Energy optimization recommendations

Mobile Support:
  - Progressive Web App (PWA)
  - Responsive design for tablets
  - Offline capability for critical functions
  - Push notifications for alerts
```

---

## ⚡ Performance & Scalability Specifications

### Target Performance Metrics
```yaml
Response Times:
  - User Interface: <2 seconds
  - Agent Communication: <100ms
  - API Calls: <500ms
  - Database Queries: <50ms
  - LLM Inference: <5 seconds

Throughput:
  - Concurrent Users: 1,000+
  - API Requests: 10,000 req/sec
  - IoT Data Points: 100,000 points/sec
  - Agent Messages: 50,000 messages/sec

Availability:
  - System Uptime: 99.9% (8.76 hours downtime/year)
  - Data Durability: 99.999999999% (11 9's)
  - Disaster Recovery: RTO <4 hours, RPO <15 minutes

Scalability:
  - Buildings Supported: 10,000+
  - Data Points: 1 billion+ per day
  - Agent Instances: 100+ per tier
  - Geographic Distribution: Multi-region deployment
```

### Cost Analysis
```yaml
Infrastructure Costs (Annual):
  - Hardware: $120,000 - $190,000 (initial)
  - Cloud Services: $36,000 - $48,000
  - Software Licenses: $24,000 - $36,000
  - Operational: $48,000 - $72,000
  - Total: $228,000 - $346,000

ROI Projections:
  - Energy Savings: 15-25% reduction
  - Operational Efficiency: 20-30% improvement
  - Payback Period: 18-24 months
  - 5-Year NPV: $2.5M - $4.2M
```

Kiến trúc này đảm bảo hệ thống EAIO có thể hoạt động hiệu quả với local LLMs, đáp ứng các yêu cầu real-time của energy management, và có khả năng mở rộng cho tương lai.