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

## Layer 3: MCP Integration Layer

### Model Context Protocol Framework
```python
# MCP Server Configuration
class MCPIntegrationLayer:
    """Comprehensive MCP integration for EAIO system"""
    
    def __init__(self):
        self.servers = {
            "energy_data_server": {
                "command": "python",
                "args": ["-m", "eaio_mcp.energy_server"],
                "env": {"DATA_PATH": "./data/energy", "DB_URL": "postgresql://..."},
                "timeout": 30,
                "tools": ["get_energy_consumption", "get_sensor_readings", "detect_anomalies"]
            },
            
            "building_control_server": {
                "command": "uvx", 
                "args": ["eaio-building-control-mcp"],
                "env": {"CONTROL_ENDPOINT": "http://localhost:8081"},
                "timeout": 60,
                "tools": ["adjust_hvac_settings", "optimize_lighting", "schedule_equipment"]
            },
            
            "weather_integration_server": {
                "command": "npx",
                "args": ["@eaio/weather-mcp-server"],
                "env": {"WEATHER_API_KEY": os.getenv("WEATHER_API_KEY")},
                "timeout": 15,
                "tools": ["get_weather_forecast", "get_historical_weather", "calculate_weather_impact"]
            },
            
            "ml_models_server": {
                "command": "python",
                "args": ["-m", "eaio_mcp.ml_server"],
                "env": {"MODEL_PATH": "./models/", "MILVUS_URI": "http://localhost:19530"},
                "timeout": 45,
                "tools": ["forecast_energy_usage", "calculate_efficiency", "recommend_optimizations"]
            },
            
            "bdg2_data_server": {
                "command": "python", 
                "args": ["-m", "eaio_mcp.bdg2_server"],
                "env": {"BDG2_DB": "postgresql://localhost/bdg2", "CACHE_TTL": "300"},
                "timeout": 20,
                "tools": ["query_bdg2_buildings", "get_benchmarking_data", "compare_performance"]
            }
        }
        
        self.tool_categories = {
            "data_collection": [
                "get_energy_consumption", "get_sensor_readings", "query_bdg2_buildings",
                "get_weather_forecast", "get_historical_weather"
            ],
            "analysis": [
                "detect_anomalies", "forecast_energy_usage", "calculate_efficiency",
                "calculate_weather_impact", "get_benchmarking_data", "compare_performance"
            ],
            "control": [
                "adjust_hvac_settings", "optimize_lighting", "schedule_equipment"
            ],
            "recommendations": [
                "recommend_optimizations", "suggest_maintenance", "calculate_roi"
            ]
        }

    async def setup_mcp_tools(self, agent_type: str) -> List[MCPTool]:
        """Setup MCP tools based on agent specialization"""
        
        if agent_type == "data_intelligence":
            server_params = [self.servers["energy_data_server"], 
                           self.servers["weather_integration_server"],
                           self.servers["bdg2_data_server"]]
                           
        elif agent_type == "optimization_strategist":
            server_params = [self.servers["ml_models_server"],
                           self.servers["building_control_server"]]
                           
        elif agent_type == "forecast_intelligence": 
            server_params = [self.servers["ml_models_server"],
                           self.servers["weather_integration_server"]]
                           
        elif agent_type == "control_coordination":
            server_params = [self.servers["building_control_server"],
                           self.servers["energy_data_server"]]
        
        # Create MCP tool instances
        tools = []
        for server_config in server_params:
            mcp_tools = await create_mcp_server_tools(
                StdioServerParams(**server_config)
            )
            tools.extend(mcp_tools)
            
        return tools

# MCP Resource Management
class MCPResourceManager:
    """Manage MCP resources and caching for performance"""
    
    def __init__(self):
        self.cache = Redis(host='localhost', port=6379, db=1)
        self.cache_ttl = {
            "weather": 1800,  # 30 minutes
            "energy_readings": 300,  # 5 minutes  
            "building_metadata": 3600,  # 1 hour
            "forecasts": 900,  # 15 minutes
            "bdg2_data": 7200  # 2 hours
        }
    
    async def cached_mcp_call(self, tool_name: str, **kwargs) -> Dict:
        """Execute MCP tool call with intelligent caching"""
        
        # Generate cache key
        cache_key = f"mcp:{tool_name}:{hash(str(sorted(kwargs.items())))}"
        
        # Check cache first
        cached_result = await self.cache.get(cache_key)
        if cached_result:
            return json.loads(cached_result)
        
        # Execute MCP tool call
        result = await execute_mcp_tool(tool_name, **kwargs)
        
        # Cache result with appropriate TTL
        category = tool_name.split('_')[0]
        ttl = self.cache_ttl.get(category, 300)
        await self.cache.setex(cache_key, ttl, json.dumps(result))
        
        return result
```

## Layer 4: Multi-Agent Framework (LangGraph + LangChain)

### LangGraph Agent Orchestration
```python
from langgraph.graph import StateGraph, END
from langgraph.prebuilt import ToolExecutor
from langchain.memory import ConversationBufferWindowMemory
from langchain.agents import AgentExecutor
from langsmith import Client as LangSmithClient

class EAIOAgentFramework:
    """LangGraph-based multi-agent framework for EAIO"""
    
    def __init__(self):
        self.langsmith_client = LangSmithClient()
        self.graph = self._build_agent_graph()
        self.memory_manager = EAIOMemoryManager()
        
    def _build_agent_graph(self) -> StateGraph:
        """Build LangGraph workflow for EAIO agents"""
        
        # Define agent state
        class AgentState(TypedDict):
            messages: List[BaseMessage]
            building_context: Dict
            analysis_results: Dict
            recommendations: List[Dict]
            user_role: str
            conversation_id: str
            
        # Create state graph
        workflow = StateGraph(AgentState)
        
        # Add agent nodes
        workflow.add_node("coordinator", self._coordinator_agent)
        workflow.add_node("data_intelligence", self._data_intelligence_agent)  
        workflow.add_node("optimization_strategist", self._optimization_strategist_agent)
        workflow.add_node("forecast_intelligence", self._forecast_intelligence_agent)
        workflow.add_node("control_coordination", self._control_coordination_agent)
        workflow.add_node("response_synthesizer", self._response_synthesizer_agent)
        
        # Define workflow edges
        workflow.set_entry_point("coordinator")
        
        workflow.add_conditional_edges(
            "coordinator",
            self._route_to_specialist,
            {
                "data_analysis": "data_intelligence",
                "optimization": "optimization_strategist", 
                "forecasting": "forecast_intelligence",
                "control": "control_coordination",
                "synthesis": "response_synthesizer"
            }
        )
        
        # Specialist agents route to synthesizer
        for agent in ["data_intelligence", "optimization_strategist", 
                     "forecast_intelligence", "control_coordination"]:
            workflow.add_edge(agent, "response_synthesizer")
            
        workflow.add_edge("response_synthesizer", END)
        
        return workflow.compile()
    
    async def _coordinator_agent(self, state: AgentState) -> AgentState:
        """Central coordinator agent using LangChain with MCP tools"""
        
        # Setup LangChain agent with MCP tools
        mcp_tools = await self.mcp_layer.setup_mcp_tools("coordinator")
        
        # Create LangChain agent with memory
        memory = ConversationBufferWindowMemory(
            k=10,
            return_messages=True,
            memory_key="chat_history"
        )
        
        agent = create_structured_chat_agent(
            llm=self.hybrid_llm_router.get_llm("coordinator"),
            tools=mcp_tools,
            memory=memory,
            verbose=True
        )
        
        # Execute with LangSmith tracing
        with self.langsmith_client.trace("coordinator_analysis"):
            result = await agent.ainvoke({
                "input": state["messages"][-1].content,
                "building_context": state["building_context"],
                "user_role": state["user_role"]
            })
        
        # Update state
        state["messages"].append(AIMessage(content=result["output"]))
        return state

    async def _data_intelligence_agent(self, state: AgentState) -> AgentState:
        """Data Intelligence Agent with BDG2 integration"""
        
        # Setup specialized tools for data analysis
        mcp_tools = await self.mcp_layer.setup_mcp_tools("data_intelligence")
        
        # Long-term memory for building patterns
        building_memory = await self.memory_manager.get_building_memory(
            state["building_context"]["building_id"]
        )
        
        # Create specialized LangChain agent
        agent = create_json_agent(
            llm=self.hybrid_llm_router.get_llm("data_analysis"),
            tools=mcp_tools,
            verbose=True
        )
        
        with self.langsmith_client.trace("data_intelligence_analysis"):
            result = await agent.ainvoke({
                "input": f"""
                Analyze energy consumption for building {state['building_context']['building_id']}.
                Consider historical patterns from BDG2 data and current readings.
                Building memory: {building_memory}
                """,
                "building_context": state["building_context"]
            })
        
        # Store analysis results
        state["analysis_results"]["data_intelligence"] = result
        
        # Update long-term memory
        await self.memory_manager.update_building_memory(
            state["building_context"]["building_id"],
            result["analysis"]
        )
        
        return state

# LangChain Memory Integration
class EAIOMemoryManager:
    """Advanced memory management for EAIO agents"""
    
    def __init__(self):
        self.milvus_client = MilvusClient(uri="http://localhost:19530")
        self.short_term_memory = {}  # Redis-backed
        self.embeddings = OpenAIEmbeddings()
        
    async def get_building_memory(self, building_id: str) -> Dict:
        """Retrieve long-term memory for specific building"""
        
        # Query Milvus for building-specific patterns
        query_vector = await self.embeddings.aembed_query(f"building_{building_id}_patterns")
        
        results = self.milvus_client.search(
            collection_name="building_memory",
            data=[query_vector],
            filter=f'building_id == "{building_id}"',
            limit=5,
            output_fields=["memory_content", "timestamp", "confidence"]
        )
        
        return {
            "historical_patterns": [r["entity"]["memory_content"] for r in results[0]],
            "last_updated": max([r["entity"]["timestamp"] for r in results[0]] or [None])
        }
    
    async def update_building_memory(self, building_id: str, analysis: Dict):
        """Update long-term memory with new insights"""
        
        # Create embedding for new insight
        insight_text = f"Building {building_id}: {analysis['summary']}"
        embedding = await self.embeddings.aembed_query(insight_text)
        
        # Store in Milvus
        self.milvus_client.insert(
            collection_name="building_memory",
            data=[{
                "id": f"{building_id}_{int(time.time())}",
                "building_id": building_id,
                "memory_content": insight_text,
                "embedding": embedding,
                "timestamp": time.time(),
                "confidence": analysis.get("confidence", 0.8)
            }]
        )
    
    def get_short_term_memory(self, conversation_id: str) -> ConversationBufferWindowMemory:
        """Get conversation-specific short-term memory"""
        
        if conversation_id not in self.short_term_memory:
            self.short_term_memory[conversation_id] = ConversationBufferWindowMemory(
                k=20,  # Keep last 20 exchanges
                return_messages=True,
                memory_key="chat_history"
            )
            
        return self.short_term_memory[conversation_id]

# LangSmith Integration for Monitoring
class LangSmithMonitoring:
    """LangSmith integration for agent performance monitoring"""
    
    def __init__(self):
        self.client = LangSmithClient()
        self.project_name = "eaio-multi-agent"
        
    async def trace_agent_interaction(self, agent_name: str, input_data: Dict, output_data: Dict):
        """Trace individual agent interactions"""
        
        with self.client.trace(
            name=f"{agent_name}_interaction",
            project_name=self.project_name,
            inputs=input_data,
            outputs=output_data
        ) as trace:
            # Add custom metadata
            trace.metadata = {
                "agent_type": agent_name,
                "building_id": input_data.get("building_context", {}).get("building_id"),
                "user_role": input_data.get("user_role"),
                "response_time": output_data.get("response_time"),
                "token_usage": output_data.get("token_usage", {})
            }
            
    async def log_multi_agent_workflow(self, workflow_id: str, state_history: List[Dict]):
        """Log complete multi-agent workflow execution"""
        
        self.client.create_run(
            name="multi_agent_workflow",
            project_name=self.project_name,
            inputs={"workflow_id": workflow_id},
            outputs={"final_state": state_history[-1]},
            run_type="chain",
            extra={
                "state_transitions": len(state_history),
                "agents_involved": list(set([s.get("current_agent") for s in state_history])),
                "total_duration": state_history[-1]["timestamp"] - state_history[0]["timestamp"]
            }
        )
```

## Integration Points

### Agent-MCP Integration Matrix
| Agent Type | Primary MCP Tools | Memory Type | LLM Provider |
|------------|------------------|-------------|--------------|
| Data Intelligence | energy_data, weather, bdg2 | Long-term + Vector | Local Llama-3.2-3B |
| Optimization Strategist | ml_models, building_control | Short-term + Long-term | External DeepSeek-V3 |
| Forecast Intelligence | ml_models, weather | Vector + Time-series | Hybrid routing |
| Control Coordination | building_control, energy_data | Short-term | Local Qwen2.5-7B |

### Performance Optimization
- **MCP Tool Caching**: Redis-based caching with category-specific TTL
- **Memory Management**: Automatic memory cleanup and optimization
- **LLM Routing**: Intelligent routing based on task complexity and privacy
- **State Persistence**: LangGraph state checkpointing for reliability

### Error Handling & Resilience
- **Circuit Breaker Pattern**: Automatic fallback for failed MCP servers
- **Retry Logic**: Exponential backoff for transient failures
- **Graceful Degradation**: Partial functionality when components unavailable
- **Health Monitoring**: Continuous monitoring of all framework components 