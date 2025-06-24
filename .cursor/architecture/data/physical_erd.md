# EAIO Physical Entity Relationship Diagram
**Architecture Mode (A.*) - Physical Data Model**

## 🎯 Overview

The EAIO physical data model translates the conceptual entities into optimized database schemas, designed for high-performance time-series operations, efficient local LLM integration, and scalable multi-agent processing.

## 🗄️ Database Technology Mapping

### Primary Database Allocation
```yaml
InfluxDB (Time-Series): Energy consumption, sensor data, metrics
SQLite (Relational): Metadata, configuration, user management  
ChromaDB (Vector): Agent memory, embeddings, semantic search
Redis (Cache): Session data, real-time metrics, message queues
```

## 📊 InfluxDB Schema Design

### Measurement: energy_consumption
```sql
-- Time-series data optimized for M1 performance
CREATE MEASUREMENT energy_consumption (
  -- Tags (indexed)
  building_id        STRING,
  meter_type         STRING,    -- electricity, water, gas, steam, hotwater, chilledwater
  location           STRING,    -- building location for regional analysis
  building_type      STRING,    -- office, retail, education, etc.
  
  -- Fields (values)
  value              FLOAT,     -- consumption value
  unit               STRING,    -- kWh, gallons, therms, etc.
  quality_score      FLOAT,     -- data quality indicator (0-1)
  baseline_value     FLOAT,     -- expected normal value
  deviation_percent  FLOAT,     -- percentage deviation from baseline
  
  -- Timestamp (automatic)
  time               TIMESTAMP  -- RFC3339 format with nanosecond precision
)

-- Retention Policy for M1 Memory Optimization
CREATE RETENTION POLICY "detailed" ON "eaio_db" DURATION 90d REPLICATION 1 DEFAULT
CREATE RETENTION POLICY "aggregated" ON "eaio_db" DURATION 5y REPLICATION 1

-- Continuous Queries for Automatic Aggregation
CREATE CONTINUOUS QUERY "hourly_aggregation" ON "eaio_db"
BEGIN
  SELECT mean(value) AS avg_consumption,
         max(value) AS peak_consumption,
         min(value) AS min_consumption,
         count(value) AS sample_count,
         stddev(value) AS consumption_variance
  INTO "eaio_db"."aggregated"."energy_hourly"
  FROM "eaio_db"."detailed"."energy_consumption"
  GROUP BY time(1h), building_id, meter_type
END
```

### Measurement: weather_data
```sql
CREATE MEASUREMENT weather_data (
  -- Tags
  location           STRING,    -- geographic location
  source             STRING,    -- weather data provider
  
  -- Fields
  temperature        FLOAT,     -- celsius
  humidity           FLOAT,     -- percentage
  wind_speed         FLOAT,     -- m/s
  precipitation      FLOAT,     -- mm
  cloud_cover        FLOAT,     -- percentage
  pressure           FLOAT,     -- hPa
  solar_radiation    FLOAT,     -- W/m²
  
  time               TIMESTAMP
)
```

### Measurement: agent_activities
```sql
CREATE MEASUREMENT agent_activities (
  -- Tags
  agent_id           STRING,
  agent_type         STRING,    -- data_manager, optimizer, forecaster, controller
  building_id        STRING,
  activity_type      STRING,    -- analysis, recommendation, control, alert
  
  -- Fields
  execution_time_ms  INTEGER,   -- performance monitoring
  success            BOOLEAN,
  confidence_score   FLOAT,     -- agent confidence in action/decision
  resource_usage_mb  INTEGER,   -- memory usage tracking
  
  time               TIMESTAMP
)
```

## 🗃️ SQLite Schema Design

### Core Metadata Tables
```sql
-- Buildings master data
CREATE TABLE buildings (
    id                  TEXT PRIMARY KEY,
    name                TEXT NOT NULL,
    location            TEXT NOT NULL,
    building_type       TEXT NOT NULL,
    size_sqft          INTEGER,
    floors             INTEGER,
    built_year         INTEGER,
    primary_use        TEXT,
    occupancy_hours    TEXT,
    energy_sources     JSON,    -- Array of energy types
    timezone           TEXT DEFAULT 'UTC',
    metadata           JSON,    -- Flexible metadata storage
    created_at         DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at         DATETIME DEFAULT CURRENT_TIMESTAMP,
    
    -- Optimization indexes
    CONSTRAINT valid_building_type CHECK (building_type IN (
        'office', 'retail', 'education', 'healthcare', 
        'hospitality', 'warehouse', 'manufacturing', 'residential'
    ))
);

-- Index for performance
CREATE INDEX idx_buildings_location ON buildings(location);
CREATE INDEX idx_buildings_type ON buildings(building_type);
CREATE INDEX idx_buildings_size ON buildings(size_sqft);

-- Energy meter configuration
CREATE TABLE energy_meters (
    id                  TEXT PRIMARY KEY,
    building_id         TEXT NOT NULL,
    meter_type          TEXT NOT NULL,
    location_in_building TEXT,
    manufacturer        TEXT,
    model              TEXT,
    installation_date   DATE,
    calibration_date    DATE,
    sampling_rate_sec   INTEGER DEFAULT 900, -- 15 minutes
    unit               TEXT NOT NULL,
    multiplier         REAL DEFAULT 1.0,
    is_active          BOOLEAN DEFAULT TRUE,
    
    FOREIGN KEY (building_id) REFERENCES buildings(id),
    CONSTRAINT valid_meter_type CHECK (meter_type IN (
        'electricity', 'water', 'gas', 'steam', 'hotwater', 'chilledwater'
    ))
);

CREATE INDEX idx_meters_building ON energy_meters(building_id);
CREATE INDEX idx_meters_type ON energy_meters(meter_type);

-- Agent configuration and state
CREATE TABLE agents (
    id                  TEXT PRIMARY KEY,
    agent_type          TEXT NOT NULL,
    name               TEXT NOT NULL,
    description        TEXT,
    model_name         TEXT,           -- LLM model used
    is_active          BOOLEAN DEFAULT TRUE,
    configuration      JSON,           -- Agent-specific config
    resource_limits    JSON,           -- Memory, CPU limits
    tools_enabled      JSON,           -- Available MCP tools
    created_at         DATETIME DEFAULT CURRENT_TIMESTAMP,
    last_active_at     DATETIME,
    
    CONSTRAINT valid_agent_type CHECK (agent_type IN (
        'data_intelligence', 'optimization_strategist', 
        'forecast_intelligence', 'control_coordination'
    ))
);

-- Agent performance metrics
CREATE TABLE agent_performance (
    id                  INTEGER PRIMARY KEY AUTOINCREMENT,
    agent_id           TEXT NOT NULL,
    metric_date        DATE NOT NULL,
    tasks_completed    INTEGER DEFAULT 0,
    avg_response_time_ms INTEGER,
    success_rate       REAL,           -- 0.0 to 1.0
    memory_usage_mb    INTEGER,
    model_swaps        INTEGER DEFAULT 0,
    error_count        INTEGER DEFAULT 0,
    
    FOREIGN KEY (agent_id) REFERENCES agents(id),
    UNIQUE(agent_id, metric_date)
);

-- User management and roles
CREATE TABLE users (
    id                  TEXT PRIMARY KEY,
    username           TEXT UNIQUE NOT NULL,
    email              TEXT UNIQUE NOT NULL,
    password_hash      TEXT NOT NULL,
    role               TEXT NOT NULL,
    full_name          TEXT,
    department         TEXT,
    building_access    JSON,           -- Array of building IDs
    preferences        JSON,           -- UI and notification preferences
    is_active          BOOLEAN DEFAULT TRUE,
    created_at         DATETIME DEFAULT CURRENT_TIMESTAMP,
    last_login_at      DATETIME,
    
    CONSTRAINT valid_role CHECK (role IN (
        'facility_manager', 'energy_analyst', 'executive', 'admin'
    ))
);
```

### Operational Tables
```sql
-- Anomaly detection results
CREATE TABLE anomalies (
    id                  INTEGER PRIMARY KEY AUTOINCREMENT,
    building_id         TEXT NOT NULL,
    meter_type          TEXT NOT NULL,
    detected_at         DATETIME NOT NULL,
    anomaly_type        TEXT NOT NULL,
    severity           TEXT NOT NULL,
    description        TEXT,
    actual_value       REAL,
    expected_value     REAL,
    deviation_percent  REAL,
    detection_method   TEXT,           -- rule_based, ml_model, etc.
    is_resolved        BOOLEAN DEFAULT FALSE,
    resolved_at        DATETIME,
    resolution_notes   TEXT,
    false_positive     BOOLEAN DEFAULT FALSE,
    
    FOREIGN KEY (building_id) REFERENCES buildings(id),
    CONSTRAINT valid_severity CHECK (severity IN ('low', 'medium', 'high', 'critical')),
    CONSTRAINT valid_anomaly_type CHECK (anomaly_type IN (
        'spike', 'drop', 'drift', 'missing_data', 'quality_issue'
    ))
);

CREATE INDEX idx_anomalies_building_time ON anomalies(building_id, detected_at);
CREATE INDEX idx_anomalies_severity ON anomalies(severity) WHERE is_resolved = FALSE;

-- Optimization recommendations
CREATE TABLE recommendations (
    id                  INTEGER PRIMARY KEY AUTOINCREMENT,
    building_id         TEXT NOT NULL,
    created_by_agent    TEXT NOT NULL,
    created_at          DATETIME DEFAULT CURRENT_TIMESTAMP,
    title              TEXT NOT NULL,
    description        TEXT NOT NULL,
    category           TEXT NOT NULL,
    priority           TEXT NOT NULL,
    estimated_savings_annually REAL,
    implementation_cost REAL,
    roi_payback_months INTEGER,
    complexity         TEXT,           -- low, medium, high
    implementation_plan JSON,
    status             TEXT DEFAULT 'pending',
    approved_by        TEXT,
    approved_at        DATETIME,
    implemented_at     DATETIME,
    actual_savings     REAL,
    notes              TEXT,
    
    FOREIGN KEY (building_id) REFERENCES buildings(id),
    FOREIGN KEY (created_by_agent) REFERENCES agents(id),
    CONSTRAINT valid_priority CHECK (priority IN ('low', 'medium', 'high', 'critical')),
    CONSTRAINT valid_status CHECK (status IN (
        'pending', 'approved', 'rejected', 'implemented', 'monitored'
    ))
);

-- Energy forecasts
CREATE TABLE forecasts (
    id                  INTEGER PRIMARY KEY AUTOINCREMENT,
    building_id         TEXT NOT NULL,
    meter_type          TEXT NOT NULL,
    created_by_agent    TEXT NOT NULL,
    created_at          DATETIME DEFAULT CURRENT_TIMESTAMP,
    forecast_type       TEXT NOT NULL,      -- hourly, daily, weekly, monthly
    forecast_horizon_hours INTEGER NOT NULL,
    model_type          TEXT,               -- linear, arima, lstm, etc.
    model_parameters    JSON,
    accuracy_score      REAL,               -- Historical accuracy
    confidence_interval REAL,               -- 0.95 for 95% confidence
    
    FOREIGN KEY (building_id) REFERENCES buildings(id),
    FOREIGN KEY (created_by_agent) REFERENCES agents(id)
);

-- Forecast data points
CREATE TABLE forecast_data (
    forecast_id         INTEGER NOT NULL,
    timestamp_future   DATETIME NOT NULL,
    predicted_value    REAL NOT NULL,
    lower_bound        REAL,
    upper_bound        REAL,
    confidence_score   REAL,
    
    FOREIGN KEY (forecast_id) REFERENCES forecasts(id),
    PRIMARY KEY (forecast_id, timestamp_future)
);
```

## 🧠 ChromaDB Collections Schema

### Agent Memory Collection
```python
# Agent memory and learning storage
agent_memory_collection = {
    "name": "agent_memory",
    "metadata": {
        "description": "Long-term memory storage for AI agents",
        "embedding_model": "all-MiniLM-L6-v2",
        "vector_size": 384
    },
    "documents": [
        {
            "id": "mem_{agent_id}_{timestamp}",
            "content": "Natural language description of learned pattern or decision",
            "metadata": {
                "agent_id": "data_intelligence_001",
                "agent_type": "data_intelligence", 
                "memory_type": "pattern_recognition",
                "building_id": "bldg_001",
                "timestamp": "2024-01-15T10:30:00Z",
                "confidence": 0.95,
                "context_tags": ["anomaly_detection", "hvac_system"],
                "success_outcome": True
            }
        }
    ]
}

# Building knowledge base
building_knowledge_collection = {
    "name": "building_knowledge",
    "metadata": {
        "description": "Semantic knowledge about buildings and energy systems",
        "embedding_model": "all-MiniLM-L6-v2"
    },
    "documents": [
        {
            "id": "know_{building_id}_{knowledge_type}",
            "content": "Descriptive knowledge about building characteristics or patterns",
            "metadata": {
                "building_id": "bldg_001",
                "knowledge_type": "operational_pattern",
                "category": "hvac_behavior",
                "reliability_score": 0.88,
                "last_updated": "2024-01-15T10:30:00Z",
                "source": "pattern_analysis",
                "validation_count": 15
            }
        }
    ]
}

# Conversation context
conversation_context_collection = {
    "name": "conversation_context", 
    "metadata": {
        "description": "User conversation history and context",
        "embedding_model": "all-MiniLM-L6-v2"
    },
    "documents": [
        {
            "id": "conv_{user_id}_{session_id}_{turn}",
            "content": "User query and agent response context",
            "metadata": {
                "user_id": "user_123",
                "session_id": "sess_456", 
                "turn_number": 3,
                "user_role": "facility_manager",
                "timestamp": "2024-01-15T10:30:00Z",
                "intent": "energy_optimization_inquiry",
                "satisfaction_score": 4.5
            }
        }
    ]
}
```

## 📦 Redis Data Structures

### Cache Schemas
```redis
# Real-time building metrics (Hash)
HSET building:{building_id}:current
  electricity_kw "245.7"
  water_gpm "12.3"  
  gas_therms "8.9"
  last_update "2024-01-15T10:30:00Z"
  alert_status "normal"
  
# Agent status tracking (Hash)
HSET agent:{agent_id}:status
  state "active"
  current_task "anomaly_detection"
  memory_usage_mb "1024"
  last_heartbeat "2024-01-15T10:30:00Z"
  model_loaded "llama3.2:3b"
  
# Session management (Hash with TTL)
HSET session:{session_id}
  user_id "user_123"
  role "facility_manager"
  building_access "[\"bldg_001\", \"bldg_002\"]"
  created_at "2024-01-15T10:00:00Z"
EXPIRE session:{session_id} 3600

# Real-time alerts queue (List)
LPUSH alerts:building:{building_id}
  "{\"type\":\"anomaly\",\"severity\":\"high\",\"timestamp\":\"2024-01-15T10:30:00Z\",\"message\":\"Electricity consumption 40% above normal\"}"
  
# Agent communication (Pub/Sub)
PUBLISH agent:coordination:optimization
  "{\"event\":\"anomaly_detected\",\"building_id\":\"bldg_001\",\"agent_id\":\"data_intel_001\",\"severity\":\"high\"}"
```

## 🔍 Query Optimization Strategies

### InfluxDB Query Patterns
```sql
-- Optimized for M1 performance - Use time boundaries
SELECT mean(value) as avg_consumption
FROM energy_consumption 
WHERE building_id = 'bldg_001' 
  AND meter_type = 'electricity'
  AND time >= now() - 24h
GROUP BY time(1h)

-- Multi-building comparison with tags
SELECT mean(value) as avg_consumption
FROM energy_consumption
WHERE building_type = 'office'
  AND time >= now() - 7d  
GROUP BY building_id, time(1d)
```

### SQLite Query Optimization
```sql
-- Efficient anomaly detection lookup
SELECT a.*, b.name as building_name
FROM anomalies a
JOIN buildings b ON a.building_id = b.id
WHERE a.is_resolved = FALSE
  AND a.severity IN ('high', 'critical')
  AND a.detected_at >= datetime('now', '-7 days')
ORDER BY a.detected_at DESC
LIMIT 50;

-- Agent performance analytics  
SELECT 
  agent_type,
  AVG(avg_response_time_ms) as avg_response,
  AVG(success_rate) as avg_success_rate,
  SUM(tasks_completed) as total_tasks
FROM agent_performance ap
JOIN agents a ON ap.agent_id = a.id
WHERE ap.metric_date >= date('now', '-30 days')
GROUP BY agent_type
ORDER BY avg_success_rate DESC;
```

## 📊 Performance Tuning for M1 Architecture

### Memory Optimization
```yaml
InfluxDB Configuration:
  cache-max-memory-size: 1073741824  # 1GB
  cache-snapshot-memory-size: 26214400  # 25MB
  max-concurrent-compactions: 2
  
SQLite Pragmas:
  PRAGMA cache_size = -262144;  # 256MB cache
  PRAGMA journal_mode = WAL;
  PRAGMA synchronous = NORMAL;
  PRAGMA temp_store = MEMORY;
  
ChromaDB Settings:
  max_batch_size: 5000
  persist_directory: "./chroma_db"
  
Redis Configuration:
  maxmemory: 512mb
  maxmemory-policy: allkeys-lru
```

### Index Strategy
```sql
-- SQLite covering indexes for common queries
CREATE INDEX idx_anomalies_building_unresolved 
ON anomalies(building_id, is_resolved, detected_at) 
WHERE is_resolved = FALSE;

CREATE INDEX idx_recommendations_status_priority
ON recommendations(status, priority, created_at)
WHERE status = 'pending';

-- InfluxDB tag-based optimization
-- Tags are automatically indexed, design schema with query patterns in mind
```

This physical ERD provides an optimized foundation for high-performance energy analytics while maintaining the flexibility needed for AI agent operations and local LLM integration. 