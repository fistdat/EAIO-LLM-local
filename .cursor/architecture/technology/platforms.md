# EAIO Technology Platform Architecture
**Architecture Mode (A.*) - BDG2 Enhanced Technology Stack**

## 🎯 Executive Summary

The enhanced EAIO technology platform integrates **Milvus vector database**, **PostgreSQL with BDG2 schema**, and **Next.js + Streamlit** frontend architecture, providing enterprise-grade performance while maintaining local-first deployment principles for MacBook Pro M1 optimization.

## 💻 Updated Hardware Architecture

### MacBook Pro M1 Resource Allocation (16GB RAM)
```yaml
Enhanced Resource Strategy:
  System + macOS: 3GB (18.75%)
  Primary LLM Model: 6-8GB (37.5-50%)
  PostgreSQL + Milvus: 3GB (18.75%)  
  Next.js + Streamlit: 1GB (6.25%)
  Redis Cache: 512MB (3.125%)
  Application Logic: 2-2.5GB (12.5-15.6%)
  
Optimized Memory Management:
  - Lazy loading for Streamlit analytics
  - Connection pooling for PostgreSQL
  - Milvus index caching optimization
  - Next.js static generation for performance
```

## 🗄️ PostgreSQL Database Stack

### Enterprise PostgreSQL Configuration
```yaml
PostgreSQL Version: 15+ with Extensions
Core Extensions:
  - TimescaleDB: Time-series optimization for meter readings
  - PostGIS: Geospatial data for building locations
  - pg_stat_statements: Query performance monitoring
  - pgcrypto: Data encryption capabilities
  
Configuration for M1 MacBook Pro:
  shared_buffers: 1GB
  effective_cache_size: 2GB  
  work_mem: 64MB
  maintenance_work_mem: 256MB
  max_connections: 50
  
Performance Optimizations:
  - Partitioned tables for meter_readings by month
  - Materialized views for aggregated analytics
  - Parallel query execution enabled
  - Automatic vacuum tuning
```

### BDG2-Enhanced Database Schema
```sql
-- Enhanced buildings table with BDG2 integration
CREATE TABLE buildings (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    bdg2_site_id            INTEGER,
    bdg2_building_id        INTEGER,
    
    -- Core Building Information
    name                    VARCHAR(255) NOT NULL,
    address                 TEXT,
    city                    VARCHAR(100),
    state_province          VARCHAR(50),
    country                 VARCHAR(50),
    postal_code             VARCHAR(20),
    timezone                VARCHAR(50) NOT NULL,
    
    -- BDG2 Classification
    primary_use             building_use_type NOT NULL,
    sub_primary_use         VARCHAR(100),
    industry                industry_type,
    sub_industry            VARCHAR(100),
    
    -- Physical Characteristics  
    gross_floor_area        INTEGER, -- square feet
    floor_count             INTEGER,
    year_built              INTEGER,
    year_renovated          INTEGER,
    
    -- Energy Infrastructure
    energy_sources          TEXT[], -- Array of energy types
    has_renewable           BOOLEAN DEFAULT FALSE,
    
    -- Operational Information
    operating_hours         JSONB, -- Flexible schedule storage
    occupancy_type          VARCHAR(50),
    
    -- Geospatial (PostGIS)
    location                GEOMETRY(POINT, 4326),
    
    -- Metadata
    created_at              TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at              TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- BDG2 Compatibility
    UNIQUE(bdg2_site_id, bdg2_building_id)
);

-- Custom ENUM types
CREATE TYPE building_use_type AS ENUM (
    'Office', 'Education', 'Lodging/residential',
    'Entertainment/public assembly', 'Retail', 'Healthcare', 
    'Public services', 'Warehouse/storage', 'Food sales & service',
    'Religious worship', 'Manufacturing', 'Technology/science'
);

CREATE TYPE industry_type AS ENUM (
    'Education Services', 'Professional/Technical Services',
    'Healthcare/Social Assistance', 'Public Administration',
    'Accommodation/Food Services', 'Retail Trade', 
    'Manufacturing', 'Information Technology'
);

-- TimescaleDB hypertable for meter readings
CREATE TABLE meter_readings (
    time                    TIMESTAMP WITH TIME ZONE NOT NULL,
    meter_id                UUID NOT NULL,
    building_id             UUID NOT NULL,
    
    -- Energy Measurements
    value                   DECIMAL(12,4) NOT NULL,
    unit                    VARCHAR(10) NOT NULL DEFAULT 'kWh',
    meter_type              meter_type_enum NOT NULL,
    
    -- Data Quality
    quality_code            quality_enum DEFAULT 'GOOD',
    is_estimated            BOOLEAN DEFAULT FALSE,
    anomaly_score           DECIMAL(5,4),
    
    -- Weather Context (denormalized for performance)
    air_temperature         DECIMAL(6,2),
    humidity                DECIMAL(5,2),
    wind_speed              DECIMAL(5,2),
    cloud_coverage          DECIMAL(3,1),
    
    -- Constraints
    FOREIGN KEY (building_id) REFERENCES buildings(id),
    FOREIGN KEY (meter_id) REFERENCES energy_meters(id)
);

-- Convert to TimescaleDB hypertable
SELECT create_hypertable('meter_readings', 'time');

-- Create continuous aggregates for performance
CREATE MATERIALIZED VIEW meter_readings_hourly
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 hour', time) AS time_hour,
    meter_id,
    building_id,
    meter_type,
    AVG(value) AS avg_value,
    MAX(value) AS max_value,
    MIN(value) AS min_value,
    COUNT(*) AS sample_count
FROM meter_readings
GROUP BY time_hour, meter_id, building_id, meter_type;

CREATE MATERIALIZED VIEW meter_readings_daily  
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 day', time_hour) AS time_day,
    meter_id,
    building_id, 
    meter_type,
    AVG(avg_value) AS avg_value,
    MAX(max_value) AS peak_value,
    MIN(min_value) AS min_value,
    SUM(avg_value) AS total_consumption
FROM meter_readings_hourly
GROUP BY time_day, meter_id, building_id, meter_type;
```

## 🧠 Milvus Vector Database Integration

### Milvus Deployment Configuration
```yaml
Milvus Version: 2.3+ (Standalone Mode)
Deployment: Docker container with persistent storage
Configuration:
  Memory Limit: 2GB
  CPU Limit: 4 cores (M1 performance cores)
  Storage: 50GB SSD allocation
  
Collections Strategy:
  - building_patterns: Energy consumption patterns
  - anomaly_signatures: Anomaly detection embeddings  
  - agent_memory: AI agent learning and context
  - user_interactions: Conversation context and preferences
  
Vector Dimensions: 384 (all-MiniLM-L6-v2 embeddings)
Index Type: HNSW for optimal M1 performance
Distance Metric: COSINE similarity
```

### Milvus Schema Design
```python
from pymilvus import Collection, FieldSchema, CollectionSchema, DataType, connections

# Connect to local Milvus instance
connections.connect("default", host="localhost", port="19530")

# Building energy patterns collection
building_patterns_fields = [
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True, auto_id=True),
    FieldSchema(name="building_id", dtype=DataType.VARCHAR, max_length=100),
    FieldSchema(name="pattern_type", dtype=DataType.VARCHAR, max_length=50),
    FieldSchema(name="time_period", dtype=DataType.VARCHAR, max_length=20),
    FieldSchema(name="consumption_pattern", dtype=DataType.FLOAT_VECTOR, dim=384),
    FieldSchema(name="weather_correlation", dtype=DataType.FLOAT_VECTOR, dim=384),
    FieldSchema(name="confidence_score", dtype=DataType.DOUBLE),
    FieldSchema(name="building_type", dtype=DataType.VARCHAR, max_length=50),
    FieldSchema(name="created_timestamp", dtype=DataType.INT64),
]

building_patterns_schema = CollectionSchema(
    building_patterns_fields, 
    "Building energy consumption patterns and correlations"
)
building_patterns = Collection("building_patterns", building_patterns_schema)

# Create HNSW index for optimal search performance
index_params = {
    "metric_type": "COSINE",
    "index_type": "HNSW", 
    "params": {"M": 8, "efConstruction": 64}
}
building_patterns.create_index("consumption_pattern", index_params)
building_patterns.create_index("weather_correlation", index_params)

# Agent memory collection for persistent learning
agent_memory_fields = [
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True, auto_id=True),
    FieldSchema(name="agent_id", dtype=DataType.VARCHAR, max_length=50),
    FieldSchema(name="memory_type", dtype=DataType.VARCHAR, max_length=30),
    FieldSchema(name="context_embedding", dtype=DataType.FLOAT_VECTOR, dim=384),
    FieldSchema(name="decision_embedding", dtype=DataType.FLOAT_VECTOR, dim=384),
    FieldSchema(name="outcome_score", dtype=DataType.DOUBLE),
    FieldSchema(name="building_context", dtype=DataType.VARCHAR, max_length=100),
    FieldSchema(name="timestamp", dtype=DataType.INT64),
    FieldSchema(name="success_metric", dtype=DataType.DOUBLE),
]

agent_memory_schema = CollectionSchema(
    agent_memory_fields,
    "AI agent learning memory and decision history"
)
agent_memory = Collection("agent_memory", agent_memory_schema)
```

### Milvus Integration Service
```python
class MilvusVectorService:
    def __init__(self):
        connections.connect("default", host="localhost", port="19530")
        self.embedding_model = SentenceTransformer('all-MiniLM-L6-v2')
        
    async def store_building_pattern(
        self, 
        building_id: str, 
        consumption_data: List[float],
        weather_data: List[float]
    ):
        """Store building energy pattern in Milvus"""
        
        # Generate embeddings
        consumption_text = self.create_consumption_description(consumption_data)
        weather_text = self.create_weather_description(weather_data) 
        
        consumption_embedding = self.embedding_model.encode([consumption_text])[0]
        weather_embedding = self.embedding_model.encode([weather_text])[0]
        
        # Insert into Milvus
        entities = [{
            "building_id": building_id,
            "pattern_type": "daily_profile",
            "time_period": "24h",
            "consumption_pattern": consumption_embedding.tolist(),
            "weather_correlation": weather_embedding.tolist(), 
            "confidence_score": 0.95,
            "building_type": await self.get_building_type(building_id),
            "created_timestamp": int(time.time() * 1000)
        }]
        
        building_patterns.insert(entities)
        building_patterns.flush()
        
    async def find_similar_buildings(
        self, 
        target_building_id: str, 
        limit: int = 10
    ) -> List[Dict]:
        """Find buildings with similar energy patterns"""
        
        # Get target building pattern
        target_pattern = await self.get_building_pattern(target_building_id)
        
        # Search for similar patterns
        search_params = {"metric_type": "COSINE", "params": {"ef": 64}}
        
        results = building_patterns.search(
            data=[target_pattern.consumption_pattern],
            anns_field="consumption_pattern",
            param=search_params,
            limit=limit,
            output_fields=["building_id", "building_type", "confidence_score"]
        )
        
        return [
            {
                "building_id": hit.entity.get("building_id"),
                "building_type": hit.entity.get("building_type"),
                "similarity_score": hit.score,
                "confidence": hit.entity.get("confidence_score")
            }
            for hit in results[0]
        ]
```

## 🌐 Enhanced Frontend Architecture

### Next.js Application Architecture
```yaml
Next.js Configuration:
  Version: Next.js 14+ with App Router
  TypeScript: Strict mode enabled
  Deployment: Static generation + ISR for performance
  
Key Features:
  - Server-side rendering for dashboard pages
  - Static generation for public pages
  - API routes for backend integration
  - Real-time WebSocket integration
  - Progressive Web App capabilities
  
Directory Structure:
  app/
    ├── (dashboard)/          # Dashboard layout group
    │   ├── buildings/        # Building management pages
    │   ├── analytics/        # Analytics dashboard  
    │   ├── agents/          # Agent monitoring
    │   └── settings/        # System configuration
    ├── api/                 # API routes
    │   ├── buildings/       # Building CRUD operations
    │   ├── metrics/         # Real-time metrics
    │   ├── agents/          # Agent communication
    │   └── auth/           # Authentication
    ├── components/          # Reusable UI components
    │   ├── charts/         # Energy visualization charts
    │   ├── forms/          # Data input forms
    │   ├── layouts/        # Page layouts
    │   └── ui/             # Base UI components
    └── lib/                # Utility libraries
        ├── database/       # PostgreSQL client
        ├── milvus/         # Vector DB client
        ├── agents/         # AI agent integration
        └── auth/           # Authentication logic
```

### Streamlit Analytics Dashboard
```python
# Streamlit analytics application for advanced energy analysis
import streamlit as st
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import pandas as pd
import asyncio
from datetime import datetime, timedelta

class EAIOAnalyticsDashboard:
    def __init__(self):
        self.setup_page_config()
        self.db_client = PostgreSQLClient()
        self.vector_client = MilvusVectorService()
        
    def setup_page_config(self):
        st.set_page_config(
            page_title="EAIO Analytics Dashboard",
            page_icon="⚡",
            layout="wide",
            initial_sidebar_state="expanded"
        )
        
    def render_main_dashboard(self):
        """Main analytics dashboard with BDG2 data integration"""
        
        st.title("⚡ Energy AI Optimizer Analytics")
        st.markdown("Advanced analytics powered by BDG2 dataset and AI agents")
        
        # Sidebar controls
        with st.sidebar:
            st.header("Dashboard Controls")
            
            # Building selection with BDG2 metadata
            buildings = self.get_available_buildings()
            selected_buildings = st.multiselect(
                "Select Buildings",
                options=buildings,
                default=buildings[:3] if len(buildings) >= 3 else buildings,
                format_func=lambda x: f"{x['name']} ({x['primary_use']})"
            )
            
            # Time range selection
            time_range = st.selectbox(
                "Time Range",
                ["Last 24 Hours", "Last Week", "Last Month", "Last Quarter", "BDG2 Historical"]
            )
            
            # Analysis type
            analysis_type = st.selectbox(
                "Analysis Type",
                ["Real-time Monitoring", "Pattern Analysis", "Anomaly Detection", 
                 "BDG2 Benchmarking", "Agent Performance"]
            )
        
        # Main content area
        if analysis_type == "Real-time Monitoring":
            self.render_realtime_monitoring(selected_buildings, time_range)
        elif analysis_type == "Pattern Analysis":
            self.render_pattern_analysis(selected_buildings, time_range)
        elif analysis_type == "BDG2 Benchmarking":
            self.render_bdg2_benchmarking(selected_buildings)
        elif analysis_type == "Agent Performance":
            self.render_agent_performance()
            
    def render_realtime_monitoring(self, buildings, time_range):
        """Real-time energy monitoring dashboard"""
        
        col1, col2, col3, col4 = st.columns(4)
        
        with col1:
            total_consumption = self.get_total_consumption(buildings, time_range)
            st.metric(
                "Total Consumption", 
                f"{total_consumption:,.0f} kWh",
                delta=f"{self.get_consumption_delta(buildings):+.1%}"
            )
            
        with col2:
            active_alerts = self.get_active_alerts(buildings)
            st.metric("Active Alerts", active_alerts, delta="-2")
            
        with col3:
            efficiency_score = self.get_efficiency_score(buildings)
            st.metric("Efficiency Score", f"{efficiency_score:.1f}/10", delta="+0.3")
            
        with col4:
            cost_savings = self.get_cost_savings(buildings)
            st.metric("Today's Savings", f"${cost_savings:,.0f}", delta="+12%")
        
        # Real-time consumption chart
        st.subheader("Real-time Energy Consumption")
        consumption_data = self.get_realtime_data(buildings)
        
        fig = go.Figure()
        for building in buildings:
            building_data = consumption_data[consumption_data['building_id'] == building['id']]
            fig.add_trace(go.Scatter(
                x=building_data['timestamp'],
                y=building_data['consumption'],
                mode='lines+markers',
                name=building['name'],
                line=dict(width=2)
            ))
            
        fig.update_layout(
            title="Hourly Energy Consumption",
            xaxis_title="Time",
            yaxis_title="Consumption (kWh)",
            hovermode='x unified'
        )
        st.plotly_chart(fig, use_container_width=True)
        
    def render_bdg2_benchmarking(self, selected_buildings):
        """BDG2 dataset benchmarking analysis"""
        
        st.subheader("🏢 BDG2 Dataset Benchmarking")
        st.markdown("Compare your buildings against the Building Data Genome Project 2 dataset")
        
        for building in selected_buildings:
            with st.expander(f"📊 {building['name']} Benchmark Analysis"):
                
                # Get BDG2 peer comparison
                peers = asyncio.run(
                    self.vector_client.find_similar_buildings(building['id'])
                )
                
                col1, col2 = st.columns(2)
                
                with col1:
                    st.write("**Building Characteristics**")
                    st.write(f"Primary Use: {building['primary_use']}")
                    st.write(f"Floor Area: {building['gross_floor_area']:,} sq ft")
                    st.write(f"Year Built: {building['year_built']}")
                    
                    # BDG2 peer buildings
                    st.write("**Similar BDG2 Buildings**")
                    peer_df = pd.DataFrame(peers)
                    st.dataframe(peer_df, hide_index=True)
                
                with col2:
                    # Performance comparison chart
                    benchmark_data = self.get_benchmark_data(building['id'], peers)
                    
                    fig = go.Figure()
                    fig.add_trace(go.Bar(
                        name='Your Building',
                        x=['Energy Intensity', 'Peak Demand', 'Load Factor'],
                        y=[benchmark_data['your_building'][metric] for metric in 
                           ['energy_intensity', 'peak_demand', 'load_factor']],
                        marker_color='lightblue'
                    ))
                    fig.add_trace(go.Bar(
                        name='BDG2 Peer Average',
                        x=['Energy Intensity', 'Peak Demand', 'Load Factor'],
                        y=[benchmark_data['peer_average'][metric] for metric in 
                           ['energy_intensity', 'peak_demand', 'load_factor']],
                        marker_color='orange'
                    ))
                    
                    fig.update_layout(
                        title="Performance vs BDG2 Peers",
                        barmode='group'
                    )
                    st.plotly_chart(fig, use_container_width=True)
```

### Real-time Dashboard Integration
```typescript
// Next.js real-time dashboard component
'use client';

import { useEffect, useState } from 'react';
import { WebSocketProvider } from '@/lib/websocket';
import { EnergyMetricsChart } from '@/components/charts/EnergyMetricsChart';
import { AlertPanel } from '@/components/alerts/AlertPanel';
import { BuildingSelector } from '@/components/selectors/BuildingSelector';

interface RealTimeDashboardProps {
  initialBuildings: Building[];
}

export default function RealTimeDashboard({ initialBuildings }: RealTimeDashboardProps) {
  const [selectedBuildings, setSelectedBuildings] = useState<Building[]>(initialBuildings);
  const [realTimeData, setRealTimeData] = useState<EnergyReading[]>([]);
  const [alerts, setAlerts] = useState<Alert[]>([]);

  useEffect(() => {
    const ws = new WebSocketProvider('ws://localhost:8000/ws/realtime');
    
    ws.on('energy_reading', (data: EnergyReading) => {
      setRealTimeData(prev => [...prev.slice(-100), data]); // Keep last 100 readings
    });
    
    ws.on('anomaly_alert', (alert: Alert) => {
      setAlerts(prev => [alert, ...prev.slice(0, 9)]); // Keep last 10 alerts
    });
    
    return () => ws.disconnect();
  }, [selectedBuildings]);

  return (
    <div className="grid grid-cols-1 lg:grid-cols-3 gap-6 p-6">
      {/* Building Selection */}
      <div className="lg:col-span-3">
        <BuildingSelector
          buildings={initialBuildings}
          selected={selectedBuildings}
          onChange={setSelectedBuildings}
        />
      </div>
      
      {/* Real-time Metrics */}
      <div className="lg:col-span-2">
        <EnergyMetricsChart 
          data={realTimeData}
          buildings={selectedBuildings}
          timeRange="24h"
        />
      </div>
      
      {/* Alert Panel */}
      <div className="lg:col-span-1">
        <AlertPanel alerts={alerts} />
      </div>
    </div>
  );
}
```

## 📊 Performance Monitoring Stack

### Enhanced Monitoring Configuration
```yaml
Monitoring Stack:
  Application Metrics:
    - Prometheus: PostgreSQL and Milvus metrics
    - Grafana: Real-time dashboards
    - Custom metrics: LLM inference time, vector search latency
    
  Database Monitoring:
    - PostgreSQL: pg_stat_monitor extension
    - Milvus: Built-in metrics endpoint
    - TimescaleDB: Continuous aggregate performance
    
  Frontend Performance:
    - Next.js: Built-in performance monitoring
    - Streamlit: Custom performance tracking
    - WebSocket: Connection quality metrics
    
Performance Targets:
  - PostgreSQL query time: <100ms (95th percentile)
  - Milvus vector search: <50ms (average)
  - Next.js page load: <1s (initial), <200ms (navigation)
  - Streamlit dashboard: <3s (initial load)
  - WebSocket latency: <50ms (real-time updates)
```

This enhanced technology platform provides enterprise-grade capabilities while maintaining the local-first deployment model optimized for MacBook Pro M1 hardware constraints. 