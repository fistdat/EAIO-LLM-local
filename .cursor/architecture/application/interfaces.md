# EAIO Application Interfaces

## Overview

This document defines the component interactions, API specifications, and interface contracts for the EAIO (Energy AI Optimizer) multi-agent system. It ensures consistent communication patterns across all architectural layers.

## 🔄 Interface Architecture Principles

### 1. **Standardized Communication Patterns**
- **MCP Protocol**: Primary interface for tool/server communication
- **REST APIs**: HTTP-based interfaces for web services
- **GraphQL**: Flexible queries for complex data relationships
- **WebSocket**: Real-time bidirectional communication
- **gRPC**: High-performance internal service communication

### 2. **Interface Design Patterns**
- **Request-Response**: Synchronous operations with immediate results
- **Event-Driven**: Asynchronous notifications and state changes
- **Publisher-Subscriber**: Decoupled messaging between components
- **Circuit Breaker**: Fault tolerance and graceful degradation
- **Rate Limiting**: Resource protection and fair usage

---

## 🖥️ Layer 1: User Interface APIs

### Next.js Frontend APIs

#### Executive Dashboard API
```typescript
interface ExecutiveDashboardAPI {
  // Portfolio Overview
  getPortfolioMetrics(): Promise<PortfolioMetrics>
  getEnergyTrends(timeRange: TimeRange): Promise<EnergyTrend[]>
  getCostAnalysis(period: Period): Promise<CostAnalysis>
  
  // Strategic Insights
  getROIAnalysis(): Promise<ROIMetrics>
  getOptimizationOpportunities(): Promise<Opportunity[]>
  getBenchmarkComparison(): Promise<BenchmarkData>
}

interface PortfolioMetrics {
  totalBuildings: number
  totalEnergySavings: number
  totalCostSavings: number
  averageEfficiencyScore: number
  alertCount: number
  optimizationCount: number
}
```

#### Manager Dashboard API
```typescript
interface ManagerDashboardAPI {
  // Operational Control
  getBuildingStatus(buildingId: string): Promise<BuildingStatus>
  getSystemAlerts(): Promise<SystemAlert[]>
  getPerformanceMetrics(): Promise<PerformanceMetrics>
  
  // Control Operations
  updateBuildingSettings(settings: BuildingSettings): Promise<void>
  scheduleOptimization(request: OptimizationRequest): Promise<void>
  acknowledgeAlert(alertId: string): Promise<void>
}

interface BuildingStatus {
  buildingId: string
  currentConsumption: number
  targetConsumption: number
  systemHealth: SystemHealth[]
  activeControls: ControlStatus[]
  lastOptimization: Date
}
```

### Streamlit Analytics API
```typescript
interface StreamlitAPI {
  // Data Exploration
  getDatasetInfo(datasetId: string): Promise<DatasetInfo>
  executeNotebook(notebookId: string, params: object): Promise<NotebookResult>
  
  // Interactive Visualizations
  generateVisualization(config: VizConfig): Promise<Visualization>
  updateVisualization(vizId: string, updates: VizUpdate): Promise<void>
  
  // BDG2 Integration
  getBDG2Analytics(filters: BDG2Filters): Promise<BDG2Analytics>
  compareBuildingsBDG2(buildingIds: string[]): Promise<BDG2Comparison>
}
```

---

## 🧠 Layer 2: Hybrid LLM Infrastructure APIs

### Hybrid Router Interface
```typescript
interface HybridLLMRouter {
  // Model Selection
  selectModel(request: ModelRequest): Promise<ModelSelection>
  routeRequest(request: LLMRequest): Promise<LLMResponse>
  
  // Privacy Classification
  classifyPrivacy(content: string): Promise<PrivacyLevel>
  
  // Cost Optimization
  estimateCost(request: LLMRequest): Promise<CostEstimate>
  optimizeForCost(request: LLMRequest): Promise<OptimizedRequest>
}

interface ModelRequest {
  content: string
  taskType: 'analysis' | 'optimization' | 'control' | 'forecast'
  privacyLevel: PrivacyLevel
  maxLatency: number
  maxCost: number
}
```

---

## 🔌 Layer 3: MCP Integration Layer APIs

### Energy Data Server
```typescript
interface EnergyDataMCP {
  // Data Collection Tools
  get_energy_consumption(params: {
    buildingId: string
    startTime: Date
    endTime: Date
    meterType?: string
  }): Promise<EnergyData[]>
  
  get_sensor_readings(params: {
    buildingId: string
    sensorType: string
    limit?: number
  }): Promise<SensorReading[]>
  
  detect_anomalies(params: {
    buildingId: string
    timeWindow: number
    sensitivity: number
  }): Promise<Anomaly[]>
}
```

### Weather Integration Server
```typescript
interface WeatherMCP {
  get_weather_forecast(params: {
    location: string
    hours: number
  }): Promise<WeatherForecast[]>
  
  get_historical_weather(params: {
    location: string
    startDate: Date
    endDate: Date
  }): Promise<WeatherData[]>
  
  calculate_weather_impact(params: {
    buildingId: string
    weatherData: WeatherData[]
  }): Promise<WeatherImpact>
}
```

### Building Control Server
```typescript
interface BuildingControlMCP {
  adjust_hvac_settings(params: {
    buildingId: string
    targetTemperature: number
    schedule?: Schedule
  }): Promise<ControlResponse>
  
  optimize_lighting(params: {
    buildingId: string
    occupancySchedule: Schedule
    naturalLight?: number
  }): Promise<LightingOptimization>
  
  schedule_equipment(params: {
    buildingId: string
    equipmentType: string
    schedule: Schedule
  }): Promise<ScheduleResponse>
}
```

---

## 🤖 Layer 4: Multi-Agent Framework APIs

### LangGraph State Interface
```typescript
interface StateGraphInterface {
  // Workflow Management
  createWorkflow(definition: WorkflowDefinition): Promise<Workflow>
  executeWorkflow(workflowId: string, input: WorkflowInput): Promise<WorkflowResult>
  
  // State Management
  getWorkflowState(workflowId: string): Promise<WorkflowState>
  updateState(workflowId: string, updates: StateUpdate): Promise<void>
  
  // Checkpointing
  createCheckpoint(workflowId: string): Promise<Checkpoint>
  restoreFromCheckpoint(checkpointId: string): Promise<void>
}

interface WorkflowState {
  messages: Message[]
  buildingContext: BuildingContext
  analysisResults: AnalysisResults
  memoryContext: MemoryContext
  performanceTracking: PerformanceMetrics
}
```

### Agent Network Interface
```typescript
interface AgentInterface {
  // Agent Communication
  invoke(input: AgentInput): Promise<AgentOutput>
  stream(input: AgentInput): AsyncIterable<AgentChunk>
  
  // Agent Status
  getStatus(): Promise<AgentStatus>
  getCapabilities(): Promise<AgentCapabilities>
  
  // Memory Access
  retrieveMemory(query: MemoryQuery): Promise<MemoryResult[]>
  storeMemory(memory: MemoryItem): Promise<void>
}

// Specialized Agent Interfaces
interface CoordinatorAgent extends AgentInterface {
  orchestrateWorkflow(task: Task): Promise<WorkflowPlan>
  distributeSubtasks(subtasks: Subtask[]): Promise<TaskDistribution>
  integrateResults(results: AgentResult[]): Promise<IntegratedResult>
}

interface DataIntelligenceAgent extends AgentInterface {
  analyzeBDG2Patterns(buildingId: string): Promise<PatternAnalysis>
  recognizeAnomalies(data: EnergyData[]): Promise<AnomalyReport>
  generateInsights(analysisParams: AnalysisParams): Promise<DataInsights>
}
```

---

## 💾 Layer 6: Data Infrastructure APIs

### Database Interface
```typescript
interface DatabaseInterface {
  // PostgreSQL Operations
  query(sql: string, params?: any[]): Promise<QueryResult>
  transaction(operations: DatabaseOperation[]): Promise<TransactionResult>
  
  // TimescaleDB Operations
  insertTimeSeries(data: TimeSeriesData[]): Promise<InsertResult>
  queryTimeSeries(query: TimeSeriesQuery): Promise<TimeSeriesResult>
  
  // Connection Management
  getConnection(): Promise<DatabaseConnection>
  releaseConnection(connection: DatabaseConnection): Promise<void>
  getHealthStatus(): Promise<DatabaseHealth>
}
```

### Vector Database Interface
```typescript
interface VectorDatabaseInterface {
  // Milvus Operations
  insert(collection: string, vectors: Vector[]): Promise<InsertResult>
  search(collection: string, query: VectorQuery): Promise<SearchResult[]>
  
  // Collection Management
  createCollection(definition: CollectionDefinition): Promise<void>
  dropCollection(name: string): Promise<void>
  getCollectionInfo(name: string): Promise<CollectionInfo>
  
  // Index Management
  createIndex(collection: string, indexParams: IndexParams): Promise<void>
  optimizeIndex(collection: string): Promise<OptimizationResult>
}
```

### Cache Interface
```typescript
interface CacheInterface {
  // Redis Operations
  get(key: string): Promise<any>
  set(key: string, value: any, ttl?: number): Promise<void>
  del(key: string): Promise<boolean>
  
  // Batch Operations
  mget(keys: string[]): Promise<any[]>
  mset(entries: CacheEntry[]): Promise<void>
  
  // Performance
  getCacheStats(): Promise<CacheStats>
  flushExpired(): Promise<number>
}
```

---

## 📊 Interface Performance Standards

### Response Time Targets
| Interface Type | Target Latency | Max Latency | SLA |
|----------------|----------------|-------------|-----|
| **REST APIs** | <200ms | <1s | 99.9% |
| **GraphQL** | <300ms | <1.5s | 99.5% |
| **WebSocket** | <50ms | <200ms | 99.9% |
| **MCP Tools** | <5s | <30s | 99.0% |
| **Agent Calls** | <3s | <10s | 95.0% |
| **Memory Access** | <50ms | <200ms | 99.9% |
| **Database** | <100ms | <500ms | 99.9% |

### Throughput Requirements
| Interface | Target RPS | Peak RPS | Concurrent Users |
|-----------|------------|----------|------------------|
| **Web APIs** | 1,000 | 5,000 | 500 |
| **Analytics** | 100 | 500 | 50 |
| **Real-time** | 10,000 | 50,000 | 1,000 |
| **Agent System** | 50 | 200 | 10 |

This comprehensive interface documentation ensures consistent, reliable communication across all EAIO system components while maintaining high performance and security standards.
