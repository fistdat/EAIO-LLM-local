# EAIO Application Component Architecture
**Architecture Mode (A.*) - System Decomposition**

## 🏗️ Overview

The EAIO application architecture follows a **Multi-Agent Microservices** pattern optimized for local LLM deployment, providing modular, scalable components that leverage MCP for standardized tool integration.

## 📐 Architectural Principles

### 1. Agent-Centric Design
- Each major capability is realized through specialized AI agents
- Agents communicate through standardized MCP protocols
- Human-agent collaboration at multiple levels

### 2. Local-First Processing
- All sensitive data processing occurs locally
- Offline-capable operations for critical functions
- Cloud services used only for non-sensitive enhancements

### 3. Hardware-Aware Optimization
- MacBook Pro M1 memory and processing constraints
- Dynamic resource allocation based on workload
- Efficient model loading and unloading strategies

## 🧩 Core Component Architecture

### Layer 1: User Experience Components

#### 1.1 Conversational Interface Engine
**Component ID**: UX-001
**Responsibility**: Natural language interaction with energy experts
```typescript
interface ConversationalEngine {
  // Multi-modal input processing
  processVoiceInput(audio: AudioStream): Promise<Intent>
  processTextInput(text: string, context: UserContext): Promise<Intent>
  
  // Context-aware response generation
  generateResponse(intent: Intent, userRole: UserRole): Promise<Response>
  
  // Multi-turn dialogue management
  maintainConversationContext(sessionId: string): ConversationState
}
```

**Key Features**:
- Role-based conversation adaption (Facility Manager, Analyst, Executive)
- Multi-turn dialogue with context retention
- Local LLM processing for privacy
- Voice and text input support

#### 1.2 Dynamic Dashboard Engine
**Component ID**: UX-002
**Responsibility**: Real-time visualization and interaction
```typescript
interface DashboardEngine {
  // Real-time data visualization
  renderEnergyMetrics(buildingId: string, timeRange: TimeRange): Chart[]
  renderAnomalyAlerts(severity: AlertLevel): AlertComponent[]
  
  // Interactive exploration
  handleDrillDown(metric: string, dimensions: string[]): DetailView
  
  // Personalization
  customizeLayout(userId: string, preferences: LayoutPreferences): Layout
}
```

**Key Features**:
- Real-time updates via WebSocket connections
- Customizable layouts per user role
- Interactive data exploration
- Mobile-responsive design

### Layer 2: Hybrid LLM Infrastructure

#### 2.0 Hybrid LLM Router
**Component ID**: LLM-001
**Responsibility**: Intelligent routing between local and external LLM services
```python
class HybridLLMRouter:
    def __init__(self, config: LLMConfig):
        self.local_ollama = OllamaClient("http://localhost:11434")
        self.openai_client = OpenAI(api_key=config.openai_key)
        self.deepseek_client = OpenAI(
            api_key=config.deepseek_key,
            base_url="https://api.deepseek.com"
        )
        self.gemini_client = genai.GenerativeModel('gemini-1.5-pro')
        self.cost_tracker = CostTracker(config.monthly_budget)
        self.privacy_filter = PrivacyFilter()
        
    async def route_request(self, request: LLMRequest) -> LLMResponse:
        # Privacy-first routing
        privacy_level = await self.privacy_filter.classify(request)
        if privacy_level == "PRIVATE":
            return await self.local_ollama.generate(request)
            
        # Budget and complexity-based routing
        if request.complexity_score > 0.8 and await self.cost_tracker.has_budget():
            provider = self.select_optimal_api(request)
            try:
                response = await self.call_external_api(provider, request)
                await self.cost_tracker.record_usage(provider, response.cost)
                return response
            except Exception as e:
                # Fallback to local on API failure
                return await self.local_ollama.generate(request)
        else:
            return await self.local_ollama.generate(request)
            
    def select_optimal_api(self, request: LLMRequest) -> str:
        """Select best API based on domain, complexity, and cost"""
        if request.domain == "technical":
            return "deepseek"  # Best for technical/coding tasks
        elif request.domain == "analysis":
            return "gemini"    # Best for data analysis
        else:
            return "openai"    # Best general capability
```

**Key Features**:
- Privacy-first routing (sensitive data stays local)
- Cost optimization with budget tracking
- Automatic fallback to local LLM
- Provider selection based on task domain
- Real-time availability monitoring

### Layer 3: Intelligent Agent Network

#### 3.1 Data Intelligence Agent
**Component ID**: AGENT-001
**Responsibility**: IoT data processing and quality assurance
```python
class DataIntelligenceAgent:
    def __init__(self, mcp_tools: MCPToolRegistry, llm_router: HybridLLMRouter):
        self.tools = mcp_tools
        self.llm_router = llm_router
        
    async def process_iot_stream(self, sensor_data: SensorReading):
        # Data validation and cleaning
        validated_data = await self.tools.validate_sensor_data(sensor_data)
        
        # Real-time anomaly detection
        anomalies = await self.tools.detect_anomalies(validated_data)
        
        # Data quality assessment
        quality_score = await self.assess_data_quality(validated_data)
        
        return ProcessedData(validated_data, anomalies, quality_score)
```

**MCP Tools Used**:
- `validate_sensor_data`: Data validation and correction
- `detect_anomalies`: Real-time anomaly detection
- `calculate_quality_metrics`: Data quality assessment

#### 3.2 Optimization Strategist Agent
**Component ID**: AGENT-002  
**Responsibility**: Energy optimization strategy development
```python
class OptimizationStrategistAgent:
    def __init__(self, mcp_tools: MCPToolRegistry, llm_router: HybridLLMRouter):
        self.tools = mcp_tools
        self.llm_router = llm_router
        
    async def develop_optimization_strategy(
        self, 
        building_profile: BuildingProfile,
        consumption_patterns: EnergyPatterns,
        constraints: OptimizationConstraints
    ):
        # Multi-objective optimization
        strategies = await self.tools.generate_optimization_scenarios(
            building_profile, consumption_patterns, constraints
        )
        
        # Cost-benefit analysis
        roi_analysis = await self.tools.calculate_roi_projections(strategies)
        
        # Risk assessment
        risk_factors = await self.analyze_implementation_risks(strategies)
        
        return OptimizationPlan(strategies, roi_analysis, risk_factors)
```

**MCP Tools Used**:
- `generate_optimization_scenarios`: Multi-objective optimization algorithms
- `calculate_roi_projections`: Financial impact modeling
- `assess_implementation_risks`: Risk analysis and mitigation

#### 3.3 Forecast Intelligence Agent
**Component ID**: AGENT-003
**Responsibility**: Energy consumption prediction and scenario modeling
```python
class ForecastIntelligenceAgent:
    def __init__(self, mcp_tools: MCPToolRegistry, llm_router: HybridLLMRouter):
        self.tools = mcp_tools
        self.llm_router = llm_router
        
    async def generate_energy_forecasts(
        self,
        historical_data: TimeSeriesData,
        weather_forecasts: WeatherData,
        building_schedule: OperationalSchedule
    ):
        # Weather-correlated forecasting
        base_forecast = await self.tools.weather_correlated_forecast(
            historical_data, weather_forecasts
        )
        
        # Scenario modeling
        scenarios = await self.tools.generate_forecast_scenarios(
            base_forecast, building_schedule
        )
        
        # Confidence intervals
        confidence_bands = await self.calculate_prediction_confidence(scenarios)
        
        return ForecastResult(base_forecast, scenarios, confidence_bands)
```

#### 3.4 Control Coordination Agent
**Component ID**: AGENT-004
**Responsibility**: Safe autonomous control and human-agent coordination
```python
class ControlCoordinationAgent:
    def __init__(self, mcp_tools: MCPToolRegistry, llm_router: HybridLLMRouter):
        self.tools = mcp_tools
        self.llm_router = llm_router
        self.safety_constraints = SafetyBoundaries()
        
    async def execute_optimization_action(
        self,
        action: OptimizationAction,
        building_context: BuildingContext
    ):
        # Safety validation
        safety_check = await self.validate_action_safety(action)
        if not safety_check.is_safe:
            return SafetyVetoResult(safety_check.reasons)
            
        # Human-in-the-loop for critical actions
        if action.risk_level == "HIGH":
            approval = await self.request_human_approval(action)
            if not approval.approved:
                return HumanVetoResult(approval.reason)
        
        # Execute through MCP tools
        result = await self.tools.execute_building_control(action)
        
        # Monitor and validate outcome
        validation = await self.validate_action_outcome(result)
        
        return ControlResult(result, validation)
```

### Layer 4: MCP Integration Framework

#### 4.1 MCP Server Orchestrator
**Component ID**: MCP-001
**Responsibility**: Manage and coordinate MCP server lifecycle
```typescript
class MCPServerOrchestrator {
  private servers: Map<string, MCPServer> = new Map()
  
  async initializeServers(config: MCPConfiguration): Promise<void> {
    for (const [name, serverConfig] of Object.entries(config.servers)) {
      const server = new MCPServer(serverConfig)
      await server.start()
      this.servers.set(name, server)
    }
  }
  
  async getAvailableTools(category?: string): Promise<MCPTool[]> {
    const allTools: MCPTool[] = []
    for (const server of this.servers.values()) {
      const tools = await server.listTools()
      allTools.push(...tools.filter(tool => 
        !category || tool.category === category
      ))
    }
    return allTools
  }
}
```

#### 3.2 Tool Registry & Discovery
**Component ID**: MCP-002
**Responsibility**: Dynamic tool discovery and capability mapping
```typescript
interface ToolRegistry {
  // Tool discovery and registration
  discoverTools(): Promise<MCPTool[]>
  registerTool(tool: MCPTool): Promise<void>
  
  // Capability matching
  findToolsForCapability(capability: string): Promise<MCPTool[]>
  
  // Tool health and performance monitoring
  monitorToolHealth(): Promise<ToolHealthStatus[]>
}
```

### Layer 5: Data & Model Management

#### 5.1 Local Model Manager
**Component ID**: DATA-001
**Responsibility**: LLM model lifecycle and resource optimization
```python
class LocalModelManager:
    def __init__(self, hardware_config: HardwareConfig):
        self.available_memory = hardware_config.memory_limit
        self.loaded_models: Dict[str, LoadedModel] = {}
        
    async def load_model(self, model_name: str, priority: int) -> LoadedModel:
        # Memory availability check
        required_memory = self.get_model_memory_requirements(model_name)
        if not self.has_available_memory(required_memory):
            await self.free_memory_for_priority(priority)
            
        # Model loading with optimization
        model = await self.load_optimized_model(model_name)
        self.loaded_models[model_name] = model
        
        return model
        
    async def unload_least_used_model(self) -> str:
        # LRU-based model eviction
        least_used = min(self.loaded_models.values(), key=lambda m: m.last_used)
        await least_used.unload()
        del self.loaded_models[least_used.name]
        return least_used.name
```

#### 5.2 Time-Series Data Engine
**Component ID**: DATA-002
**Responsibility**: Optimized time-series data storage and retrieval
```python
class TimeSeriesEngine:
    def __init__(self, config: DatabaseConfig):
        self.primary_db = InfluxDBClient(config.influx)
        self.cache_layer = RedisClient(config.redis)
        self.metadata_db = PostgreSQLClient(config.postgres)
        
    async def store_energy_reading(
        self, 
        reading: EnergyReading
    ) -> StorageResult:
        # Data validation and enrichment
        validated_reading = await self.validate_reading(reading)
        
        # Primary storage
        await self.primary_db.write_point(validated_reading)
        
        # Cache hot data
        if self.is_hot_data(validated_reading):
            await self.cache_layer.store(validated_reading)
            
        # Update metadata
        await self.update_building_metadata(validated_reading)
        
        return StorageResult(success=True, reading_id=validated_reading.id)
```

#### 5.3 Agent Memory Store
**Component ID**: DATA-003
**Responsibility**: Persistent agent memory and learning storage
```python
class AgentMemoryStore:
    def __init__(self, vector_db: ChromaDB):
        self.vector_db = vector_db
        self.embedding_model = SentenceTransformer('all-MiniLM-L6-v2')
        
    async def store_agent_memory(
        self,
        agent_id: str,
        memory: AgentMemory
    ) -> str:
        # Generate embeddings for semantic search
        embedding = self.embedding_model.encode(memory.content)
        
        # Store in vector database
        memory_id = await self.vector_db.add(
            documents=[memory.content],
            embeddings=[embedding],
            metadatas=[{
                'agent_id': agent_id,
                'timestamp': memory.timestamp,
                'memory_type': memory.type
            }]
        )
        
        return memory_id
        
    async def retrieve_relevant_memories(
        self,
        agent_id: str,
        query: str,
        limit: int = 10
    ) -> List[AgentMemory]:
        query_embedding = self.embedding_model.encode([query])
        
        results = await self.vector_db.query(
            query_embeddings=query_embedding,
            where={'agent_id': agent_id},
            n_results=limit
        )
        
        return [AgentMemory.from_dict(result) for result in results]
```

## 🔗 Component Integration Patterns

### 1. Agent-to-Agent Communication
```python
# Event-driven communication via message bus
class AgentCommunicationBus:
    async def publish_event(self, event: AgentEvent):
        await self.message_broker.publish(
            topic=f"agent.{event.target_agent}.{event.event_type}",
            message=event.serialize()
        )
        
    async def subscribe_to_events(
        self, 
        agent_id: str, 
        event_types: List[str]
    ):
        for event_type in event_types:
            await self.message_broker.subscribe(
                topic=f"agent.{agent_id}.{event_type}",
                handler=self.handle_agent_event
            )
```

### 2. UI-Agent Integration
```typescript
// Real-time updates via WebSocket
class AgentUIBridge {
  private wsConnection: WebSocketConnection
  
  async subscribeToAgentUpdates(agentId: string): Promise<void> {
    await this.wsConnection.subscribe(`agent.${agentId}.updates`, (update) => {
      this.updateUI(update)
    })
  }
  
  async sendUserCommand(command: UserCommand): Promise<AgentResponse> {
    return await this.wsConnection.request('agent.command', command)
  }
}
```

### 3. MCP Tool Integration
```python
# Standardized tool invocation
class MCPToolInvoker:
    async def invoke_tool(
        self,
        tool_name: str,
        parameters: Dict[str, Any]
    ) -> ToolResult:
        server = await self.find_server_for_tool(tool_name)
        
        try:
            result = await server.call_tool(tool_name, parameters)
            return ToolResult(success=True, data=result)
        except Exception as e:
            return ToolResult(success=False, error=str(e))
```

## 📊 Component Performance Characteristics

| Component | Memory Usage | CPU Usage | Response Time | Scalability |
|-----------|-------------|-----------|---------------|-------------|
| Conversational Engine | 512MB | Low | <200ms | Horizontal |
| Dashboard Engine | 256MB | Medium | <100ms | Horizontal |
| Data Intelligence Agent | 1GB | High | <500ms | Vertical |
| Optimization Agent | 2GB | Very High | <5s | Vertical |
| Forecast Agent | 1GB | High | <2s | Vertical |
| Control Agent | 512MB | Medium | <1s | Horizontal |
| MCP Orchestrator | 256MB | Low | <50ms | Horizontal |
| Model Manager | 8GB | Variable | <10s | Vertical |
| Time-Series Engine | 1GB | Medium | <100ms | Horizontal |
| Memory Store | 512MB | Low | <200ms | Horizontal |

## 🎯 Component Deployment Strategy

### High Availability Components
- **Data Intelligence Agent**: 2x redundancy with load balancing
- **Time-Series Engine**: Master-replica configuration
- **MCP Orchestrator**: 3x cluster with leader election

### Resource-Intensive Components  
- **Optimization Agent**: On-demand loading with model caching
- **Model Manager**: Shared resource pool with request queuing
- **Forecast Agent**: Batch processing with result caching

### Critical Path Components
- **Dashboard Engine**: Edge caching with CDN
- **Conversational Engine**: Connection pooling with failover
- **Control Agent**: Hot standby with manual failover 