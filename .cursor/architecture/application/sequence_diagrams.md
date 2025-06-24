# EAIO Sequence Diagrams
**Architecture Mode (A.*) - Process Flow Design**

## 🎯 Overview

This document defines the key sequence interactions in the EAIO system, showing how business processes translate into system-level communications between agents, components, and external systems.

## 🔄 Core Business Process Sequences

### 1. Real-Time Energy Anomaly Detection & Response (BDG2 Enhanced)

```mermaid
sequenceDiagram
    participant IoT as IoT Sensors
    participant MCP as MCP Energy Server
    participant DIA as Data Intelligence Agent
    participant PG as PostgreSQL
    participant Milvus as Milvus Vector DB
    participant NextJS as Next.js Dashboard
    participant Streamlit as Streamlit Analytics
    participant CCA as Control Coordination Agent
    participant FM as Facility Manager

    IoT->>+MCP: BDG2-formatted Reading (electricity: 450kWh)
    MCP->>+DIA: validate_sensor_data(reading)
    DIA->>DIA: Data Quality Check (BDG2 standards)
    
    par Data Storage
        DIA->>+PG: INSERT INTO meter_readings (BDG2 schema)
        PG-->>-DIA: storage_confirmed
    and Pattern Analysis
        DIA->>+Milvus: search_similar_patterns(building_type, consumption)
        Milvus-->>-DIA: peer_comparison_data
    end
    
    DIA->>DIA: detect_anomalies(current_vs_bdg2_baseline)
    
    alt Anomaly Detected
        DIA->>+NextJS: WebSocket anomaly_alert(HIGH_CONSUMPTION)
        NextJS-->>-FM: Real-time Alert Notification
        
        DIA->>+Streamlit: update_analytics_dashboard(anomaly_data)
        Streamlit-->>-DIA: dashboard_updated
        
        DIA->>+CCA: request_optimization_action(reduce_consumption)
        CCA->>+Milvus: get_similar_successful_actions(building_profile)
        Milvus-->>-CCA: proven_strategies
        CCA->>CCA: validate_action_safety()
        
        alt Safe Action
            CCA->>MCP: execute_building_control(reduce_hvac_20%)
            MCP-->>Building Systems: Adjust HVAC Settings
            Building Systems-->>MCP: Control Confirmed
            MCP-->>CCA: action_executed
            CCA->>NextJS: update_action_status(EXECUTED)
        else Unsafe Action
            CCA->>FM: request_human_approval(action_details)
            FM-->>CCA: approval_decision
        end
    end
    
    DIA-->>-MCP: processing_complete
```

### 2. Strategic Energy Optimization Planning (BDG2 + Enhanced Stack)

```mermaid
sequenceDiagram
    participant EA as Energy Analyst
    participant CE as Conversational Engine
    participant OSA as Optimization Strategist Agent
    participant FIA as Forecast Intelligence Agent
    participant PG as PostgreSQL
    participant Milvus as Milvus Vector DB
    participant NextJS as Next.js Dashboard
    participant Streamlit as Streamlit Analytics

    EA->>+CE: "Generate optimization plan for Building A"
    CE->>CE: parse_intent(text, user_role=ANALYST)
    CE->>+OSA: develop_optimization_strategy(building_a)
    
    par Data Collection
        OSA->>+PG: get_building_profile(building_a) WITH BDG2 metadata
        PG-->>-OSA: building_characteristics + industry_benchmarks
    and Pattern Analysis
        OSA->>+Milvus: find_similar_buildings(building_type, size, location)
        Milvus-->>-OSA: bdg2_peer_buildings + success_patterns
    end
    
    OSA->>+PG: get_consumption_patterns(building_a, 6_months) FROM meter_readings
    PG-->>-OSA: time_series_data + weather_correlations
    
    OSA->>+FIA: generate_energy_forecasts(building_a, bdg2_scenarios)
    FIA->>+PG: get_weather_data(location, historical + forecast)
    PG-->>-FIA: weather_time_series
    FIA->>+Milvus: get_gepiii_models(building_characteristics)
    Milvus-->>-FIA: competition_proven_models
    FIA->>FIA: apply_bdg2_forecasting_models()
    FIA-->>-OSA: enhanced_forecast_results
    
    OSA->>+Milvus: search_optimization_strategies(building_profile, peer_success)
    Milvus-->>-OSA: proven_optimization_options
    
    OSA->>OSA: calculate_roi_projections(strategies, bdg2_benchmarks)
    
    OSA->>+Milvus: store_optimization_session(strategy, confidence, context)
    Milvus-->>-OSA: memory_vector_stored
    
    OSA-->>-CE: optimization_plan_complete
    
    par Dashboard Updates
        CE->>+NextJS: render_optimization_summary(plan)
        NextJS-->>-EA: Interactive Strategy Overview
    and Analytics Display
        CE->>+Streamlit: render_detailed_analytics(plan, bdg2_comparison)
        Streamlit-->>-EA: Advanced Analytics Dashboard
    end
```

### 3. Multi-Agent Collaboration for Portfolio Analysis

```mermaid
sequenceDiagram
    participant EX as Executive
    participant CE as Conversational Engine
    participant DIA as Data Intelligence Agent
    participant OSA as Optimization Strategist Agent
    participant FIA as Forecast Intelligence Agent
    participant ACB as Agent Communication Bus
    participant UI as Dashboard Engine

    EX->>+CE: "Analyze energy performance across all buildings"
    CE->>CE: parse_intent(portfolio_analysis)
    
    CE->>+ACB: broadcast_coordination_request(portfolio_analysis)
    
    ACB->>+DIA: analyze_portfolio_data_quality()
    ACB->>+OSA: identify_portfolio_optimization_opportunities()
    ACB->>+FIA: generate_portfolio_forecasts()
    
    par Data Analysis
        DIA->>DIA: aggregate_building_metrics()
        DIA->>DIA: identify_performance_outliers()
        DIA->>ACB: publish_data_insights()
    and Optimization Analysis
        OSA->>OSA: benchmark_building_performance()
        OSA->>OSA: identify_best_practices()
        OSA->>ACB: publish_optimization_insights()
    and Forecast Analysis
        FIA->>FIA: model_portfolio_trends()
        FIA->>FIA: predict_seasonal_variations()
        FIA->>ACB: publish_forecast_insights()
    end
    
    ACB->>CE: consolidate_agent_insights()
    CE->>CE: synthesize_executive_summary()
    
    CE->>+UI: render_executive_dashboard(insights)
    UI-->>-EX: Portfolio Performance Summary
```

### 4. MCP Tool Integration & Failover

```mermaid
sequenceDiagram
    participant Agent as Any Agent
    parameter MCPOrch as MCP Orchestrator
    participant MCPServer1 as MCP Energy Server
    participant MCPServer2 as MCP Backup Server
    participant ToolReg as Tool Registry
    participant HealthMon as Health Monitor

    Agent->>+MCPOrch: invoke_tool("get_energy_consumption", params)
    MCPOrch->>+ToolReg: find_server_for_tool("get_energy_consumption")
    ToolReg-->>-MCPOrch: primary_server=MCPServer1
    
    MCPOrch->>+MCPServer1: call_tool("get_energy_consumption", params)
    
    alt Server Available
        MCPServer1-->>-MCPOrch: tool_result(energy_data)
        MCPOrch-->>-Agent: tool_result(energy_data)
    else Server Unavailable
        MCPServer1-->>MCPOrch: connection_timeout
        MCPOrch->>+HealthMon: report_server_failure(MCPServer1)
        HealthMon-->>-MCPOrch: failover_recommendation
        
        MCPOrch->>+ToolReg: find_backup_server("get_energy_consumption")
        ToolReg-->>-MCPOrch: backup_server=MCPServer2
        
        MCPOrch->>+MCPServer2: call_tool("get_energy_consumption", params)
        MCPServer2-->>-MCPOrch: tool_result(energy_data)
        MCPOrch-->>-Agent: tool_result(energy_data)
        
        MCPOrch->>HealthMon: update_server_status(MCPServer1=DOWN)
    end
```

### 5. Local LLM Model Loading & Resource Management

```mermaid
sequenceDiagram
    participant Agent as Agent Request
    participant LMM as Local Model Manager
    participant OS as Operating System
    participant Model1 as Llama-3.2-3B
    participant Model2 as Qwen2.5-7B

    Agent->>+LMM: load_model("qwen2.5-7b-instruct", priority=HIGH)
    LMM->>LMM: check_memory_availability(7GB_required)
    
    alt Sufficient Memory
        LMM->>+OS: allocate_memory(7GB)
        OS-->>-LMM: memory_allocated
        LMM->>+Model2: load_model_weights()
        Model2-->>-LMM: model_ready
        LMM-->>-Agent: LoadedModel(qwen2.5-7b)
    else Insufficient Memory
        LMM->>LMM: identify_least_used_model()
        LMM->>+Model1: unload_model()
        Model1-->>-LMM: model_unloaded
        LMM->>+OS: free_memory(3GB)
        OS-->>-LMM: memory_freed
        
        LMM->>+OS: allocate_memory(7GB)
        OS-->>-LMM: memory_allocated
        LMM->>+Model2: load_model_weights()
        Model2-->>-LMM: model_ready
        LMM-->>-Agent: LoadedModel(qwen2.5-7b)
    end
```

## 🔄 Integration Patterns

### 1. Event-Driven Agent Communication
```mermaid
sequenceDiagram
    participant ProducerAgent as Producer Agent
    participant MessageBus as Message Bus
    participant ConsumerAgent1 as Consumer Agent 1
    participant ConsumerAgent2 as Consumer Agent 2

    ProducerAgent->>+MessageBus: publish_event("anomaly.detected", event_data)
    MessageBus->>MessageBus: route_to_subscribers("anomaly.detected")
    
    par Parallel Processing
        MessageBus->>+ConsumerAgent1: handle_event(anomaly_event)
        ConsumerAgent1->>ConsumerAgent1: process_security_response()
        ConsumerAgent1-->>-MessageBus: processing_complete
    and
        MessageBus->>+ConsumerAgent2: handle_event(anomaly_event)
        ConsumerAgent2->>ConsumerAgent2: process_optimization_response()
        ConsumerAgent2-->>-MessageBus: processing_complete
    end
    
    MessageBus-->>-ProducerAgent: event_published
```

### 2. Conversational Context Management
```mermaid
sequenceDiagram
    participant User as User
    participant CE as Conversational Engine
    participant LLM as Local LLM
    parameter CS as Context Store
    participant MS as Memory Store

    User->>+CE: "What was the consumption trend last month?"
    CE->>+CS: get_conversation_context(session_id)
    CS-->>-CE: previous_context
    
    CE->>+MS: retrieve_relevant_memories(user_id, query)
    MS-->>-CE: relevant_memories
    
    CE->>CE: build_enhanced_prompt(query + context + memories)
    CE->>+LLM: generate_response(enhanced_prompt)
    LLM-->>-CE: response
    
    CE->>+CS: update_conversation_context(session_id, new_context)
    CS-->>-CE: context_updated
    
    CE-->>-User: "Based on your building portfolio, last month showed..."
```

## 📊 Enhanced Performance Considerations

### Sequence Timing Requirements (BDG2 + Enhanced Stack)
| Process | Target Response Time | Critical Path | Fallback Strategy |
|---------|---------------------|---------------|-------------------|
| Anomaly Detection | <3 minutes | IoT → PostgreSQL → Milvus → Alert | BDG2 baseline comparison |
| Optimization Planning | <90 minutes | Agent collaboration → BDG2 analysis → Recommendations | Pre-computed BDG2 scenarios |
| Portfolio Analysis | <20 minutes | Multi-agent coordination → Vector similarity → Synthesis | Incremental pattern updates |
| MCP Tool Failover | <5 seconds | Health check → Backup selection → Tool execution | Local PostgreSQL cache |
| Model Loading | <20 seconds | Memory management → Model initialization | Keep hot models loaded |
| Vector Search | <50ms | Milvus similarity search | Local pattern cache |
| PostgreSQL Query | <100ms | Time-series + metadata joins | Materialized views |
| Next.js Page Load | <1s | SSR + static generation | CDN + ISR |
| Streamlit Dashboard | <2s | Analytics computation | Cached computations |

### Enhanced Scalability Patterns
- **Database Optimization**: PostgreSQL read replicas, Milvus index tuning
- **Vector Similarity**: Parallel search across collections, intelligent caching
- **Frontend Performance**: Next.js ISR, Streamlit selective reruns
- **BDG2 Integration**: Batch processing, materialized views for common queries
- **Memory Management**: Intelligent model swapping, connection pooling 