# EAIO BDG2 Dataset Integration Model
**Architecture Mode (A.*) - Real Data Integration Design**

## 🎯 Overview

Integration of the [Building Data Genome Project 2 (BDG2)](https://github.com/buds-lab/building-data-genome-project-2) dataset provides EAIO with real-world energy consumption data from 3,053 energy meters across 1,636 non-residential buildings, enabling validated AI training and benchmarking against actual energy patterns.

## 📊 BDG2 Dataset Characteristics

### Dataset Scale & Scope
```yaml
Real Dataset Metrics:
  Buildings: 1,636 non-residential buildings
  Energy Meters: 3,053 total meters
  Time Range: 2016-2017 (2 full years)
  Measurement Frequency: Hourly
  Total Measurements: ~53.6 million data points
  Geographic Coverage: 19 sites across North America and Europe
  
Meter Types Distribution:
  - Electricity: Primary energy source (whole building)
  - Heating Water: HVAC heating systems
  - Cooling Water: HVAC cooling systems  
  - Steam: Industrial/commercial heating
  - Irrigation: Outdoor water systems
  - Solar: Renewable energy generation
```

### Building Categories from BDG2
```yaml
Primary Use Categories:
  - Office: Commercial office buildings
  - Education: Schools, universities, research facilities
  - Lodging/Residential: Hotels, dormitories
  - Entertainment/Public Assembly: Theaters, convention centers
  - Retail: Shopping centers, stores
  - Healthcare: Hospitals, clinics
  - Public Services: Government buildings
  - Warehouse/Storage: Distribution centers
  - Food Sales/Service: Restaurants, grocery stores
  - Religious Worship: Churches, temples
  - Manufacturing: Industrial facilities
  - Technology/Science: Data centers, laboratories

Industry Classifications:
  - Education Services
  - Professional/Technical Services  
  - Healthcare/Social Assistance
  - Public Administration
  - Accommodation/Food Services
  - Retail Trade
  - Manufacturing
  - Information Technology
```

## 🗄️ PostgreSQL Schema Integration

### BDG2-Aligned Database Schema
```sql
-- Buildings table enhanced with BDG2 metadata
CREATE TABLE buildings (
    id                      TEXT PRIMARY KEY,
    site_id                 INTEGER NOT NULL,
    building_id             INTEGER NOT NULL, -- BDG2 building ID
    
    -- BDG2 Metadata Fields
    primary_use             TEXT NOT NULL,
    sub_primary_use         TEXT,
    industry                TEXT,
    sub_industry            TEXT,
    timezone                TEXT NOT NULL,
    
    -- Physical Characteristics
    floor_count             INTEGER,
    square_feet             INTEGER,
    year_built              INTEGER,
    
    -- Energy Infrastructure
    has_electricity         BOOLEAN DEFAULT TRUE,
    has_chilledwater        BOOLEAN DEFAULT FALSE,
    has_hotwater            BOOLEAN DEFAULT FALSE,
    has_steam               BOOLEAN DEFAULT FALSE,
    has_irrigation          BOOLEAN DEFAULT FALSE,
    has_solar               BOOLEAN DEFAULT FALSE,
    
    -- BDG2 Competition Data
    in_gepiii_competition   BOOLEAN DEFAULT FALSE,
    kaggle_building_id      INTEGER,
    
    created_at              TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at              TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT valid_primary_use CHECK (primary_use IN (
        'Office', 'Education', 'Lodging/residential', 
        'Entertainment/public assembly', 'Retail', 'Healthcare',
        'Public services', 'Warehouse/storage', 'Food sales & service',
        'Religious worship', 'Manufacturing', 'Technology/science'
    ))
);

-- Energy meters table aligned with BDG2 structure
CREATE TABLE energy_meters (
    id                      TEXT PRIMARY KEY,
    building_id             TEXT NOT NULL REFERENCES buildings(id),
    site_id                 INTEGER NOT NULL,
    meter_id                INTEGER NOT NULL, -- BDG2 meter ID
    
    -- BDG2 Meter Classification
    meter_type              TEXT NOT NULL,
    
    -- Meter Specifications
    unit                    TEXT NOT NULL DEFAULT 'kWh',
    sampling_interval       INTEGER DEFAULT 3600, -- seconds (hourly)
    
    -- Data Quality Metrics
    data_completeness       DECIMAL(5,2), -- Percentage of non-null readings
    anomaly_rate           DECIMAL(5,2), -- Percentage of anomalous readings
    
    -- Calibration and Maintenance
    last_calibration       DATE,
    maintenance_schedule   TEXT,
    
    is_active              BOOLEAN DEFAULT TRUE,
    created_at             TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT valid_meter_type CHECK (meter_type IN (
        'electricity', 'chilledwater', 'hotwater', 'steam', 'irrigation', 'solar'
    )),
    UNIQUE(site_id, building_id, meter_id)
);

-- BDG2 meter readings with time-series optimization
CREATE TABLE meter_readings (
    reading_id              BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    meter_id                TEXT NOT NULL REFERENCES energy_meters(id),
    timestamp               TIMESTAMP NOT NULL,
    
    -- BDG2 Data Fields
    meter_reading           DECIMAL(12,4) NOT NULL, -- Energy consumption value
    
    -- Data Quality Indicators
    is_estimated            BOOLEAN DEFAULT FALSE,
    quality_flag            TEXT DEFAULT 'GOOD',
    anomaly_score          DECIMAL(4,3),
    
    -- Weather Correlation (from BDG2 weather data)
    air_temperature        DECIMAL(6,2),
    cloud_coverage         DECIMAL(3,1),
    dew_temperature        DECIMAL(6,2),
    precip_depth_1_hr      DECIMAL(6,2),
    sea_level_pressure     DECIMAL(8,2),
    wind_direction         DECIMAL(5,1),
    wind_speed             DECIMAL(5,2),
    
    created_at             TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT valid_quality_flag CHECK (quality_flag IN (
        'GOOD', 'ESTIMATED', 'SUSPICIOUS', 'BAD'
    ))
);

-- Indexes optimized for time-series queries
CREATE INDEX idx_meter_readings_meter_time ON meter_readings(meter_id, timestamp);
CREATE INDEX idx_meter_readings_timestamp ON meter_readings(timestamp);
CREATE INDEX idx_meter_readings_building_time ON meter_readings(
    (SELECT building_id FROM energy_meters WHERE id = meter_id), timestamp
);

-- BDG2 Weather data table
CREATE TABLE weather_data (
    id                     BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    site_id                INTEGER NOT NULL,
    timestamp              TIMESTAMP NOT NULL,
    
    -- BDG2 Weather Fields
    air_temperature        DECIMAL(6,2),
    cloud_coverage         DECIMAL(3,1),
    dew_temperature        DECIMAL(6,2),
    precip_depth_1_hr      DECIMAL(6,2),
    sea_level_pressure     DECIMAL(8,2),
    wind_direction         DECIMAL(5,1),
    wind_speed             DECIMAL(5,2),
    
    -- Derived weather metrics
    heat_index             DECIMAL(6,2),
    wind_chill             DECIMAL(6,2),
    apparent_temperature   DECIMAL(6,2),
    
    data_source            TEXT DEFAULT 'BDG2',
    created_at             TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(site_id, timestamp)
);

-- BDG2 Holiday and special events
CREATE TABLE calendar_events (
    id                     SERIAL PRIMARY KEY,
    site_id                INTEGER NOT NULL,
    event_date             DATE NOT NULL,
    event_type             TEXT NOT NULL,
    event_name             TEXT,
    is_business_day        BOOLEAN NOT NULL,
    
    CONSTRAINT valid_event_type CHECK (event_type IN (
        'HOLIDAY', 'WEEKEND', 'SPECIAL_EVENT', 'MAINTENANCE'
    ))
);
```

## 🧠 Milvus Vector Database Integration

### Agent Memory Collections for BDG2 Patterns
```python
# Milvus collections optimized for BDG2 data patterns
from pymilvus import Collection, FieldSchema, CollectionSchema, DataType

# Building energy patterns collection
building_patterns_schema = CollectionSchema([
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True),
    FieldSchema(name="building_id", dtype=DataType.VARCHAR, max_length=50),
    FieldSchema(name="pattern_type", dtype=DataType.VARCHAR, max_length=100),
    FieldSchema(name="primary_use", dtype=DataType.VARCHAR, max_length=100),
    FieldSchema(name="industry", dtype=DataType.VARCHAR, max_length=100),
    FieldSchema(name="pattern_embedding", dtype=DataType.FLOAT_VECTOR, dim=384),
    FieldSchema(name="confidence_score", dtype=DataType.DOUBLE),
    FieldSchema(name="seasonal_factor", dtype=DataType.DOUBLE),
    FieldSchema(name="timestamp", dtype=DataType.INT64)
])

building_patterns = Collection("building_energy_patterns", building_patterns_schema)

# Anomaly detection patterns from BDG2 data
anomaly_patterns_schema = CollectionSchema([
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True),
    FieldSchema(name="building_id", dtype=DataType.VARCHAR, max_length=50),
    FieldSchema(name="meter_type", dtype=DataType.VARCHAR, max_length=50),
    FieldSchema(name="anomaly_type", dtype=DataType.VARCHAR, max_length=100),
    FieldSchema(name="anomaly_embedding", dtype=DataType.FLOAT_VECTOR, dim=384),
    FieldSchema(name="severity_score", dtype=DataType.DOUBLE),
    FieldSchema(name="weather_correlation", dtype=DataType.DOUBLE),
    FieldSchema(name="detection_timestamp", dtype=DataType.INT64)
])

anomaly_patterns = Collection("anomaly_detection_patterns", anomaly_patterns_schema)

# GEPIII competition insights collection
competition_insights_schema = CollectionSchema([
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True),
    FieldSchema(name="kaggle_building_id", dtype=DataType.INT64),
    FieldSchema(name="prediction_model", dtype=DataType.VARCHAR, max_length=100),
    FieldSchema(name="insight_embedding", dtype=DataType.FLOAT_VECTOR, dim=384),
    FieldSchema(name="accuracy_score", dtype=DataType.DOUBLE),
    FieldSchema(name="feature_importance", dtype=DataType.VARCHAR, max_length=500),
    FieldSchema(name="model_timestamp", dtype=DataType.INT64)
])

competition_insights = Collection("gepiii_insights", competition_insights_schema)
```

### BDG2 Data Integration Pipeline
```python
class BDG2DataIntegrator:
    def __init__(self, postgres_conn, milvus_conn):
        self.postgres = postgres_conn
        self.milvus = milvus_conn
        self.embedding_model = SentenceTransformer('all-MiniLM-L6-v2')
        
    async def ingest_bdg2_buildings(self, bdg2_metadata_path: str):
        """Ingest BDG2 building metadata into PostgreSQL"""
        import pandas as pd
        
        # Load BDG2 metadata
        buildings_df = pd.read_csv(f"{bdg2_metadata_path}/building_meta.csv")
        
        for _, building in buildings_df.iterrows():
            await self.postgres.execute("""
                INSERT INTO buildings (
                    id, site_id, building_id, primary_use, sub_primary_use,
                    industry, sub_industry, timezone, floor_count, square_feet,
                    year_built, in_gepiii_competition
                ) VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12)
                ON CONFLICT (id) DO UPDATE SET updated_at = CURRENT_TIMESTAMP
            """, 
                f"bldg_{building['site_id']}_{building['building_id']}",
                building['site_id'], building['building_id'],
                building['primary_use'], building.get('sub_primary_use'),
                building['industry'], building.get('sub_industry'),
                building['timezone'], building.get('floor_count'),
                building.get('square_feet'), building.get('year_built'),
                building.get('in_gepiii', False)
            )
            
    async def ingest_bdg2_meter_data(self, bdg2_data_path: str):
        """Ingest BDG2 meter readings with batch processing"""
        import pandas as pd
        from pathlib import Path
        
        meter_files = Path(f"{bdg2_data_path}/meters/raw").glob("*.csv")
        
        for meter_file in meter_files:
            # Extract meter info from filename
            site_id, building_id, meter_type = self.parse_meter_filename(meter_file.name)
            
            # Create meter record
            meter_id = f"meter_{site_id}_{building_id}_{meter_type}"
            await self.postgres.execute("""
                INSERT INTO energy_meters (
                    id, building_id, site_id, meter_id, meter_type
                ) VALUES ($1, $2, $3, $4, $5)
                ON CONFLICT (id) DO NOTHING
            """, 
                meter_id, f"bldg_{site_id}_{building_id}",
                site_id, f"{building_id}_{meter_type}", meter_type
            )
            
            # Batch load meter readings
            meter_df = pd.read_csv(meter_file)
            await self.batch_insert_readings(meter_id, meter_df)
            
    async def generate_building_patterns(self, building_id: str):
        """Generate and store building energy patterns in Milvus"""
        
        # Fetch building consumption patterns
        patterns = await self.postgres.fetch("""
            SELECT 
                mr.meter_reading,
                em.meter_type,
                EXTRACT(hour FROM mr.timestamp) as hour,
                EXTRACT(dow FROM mr.timestamp) as day_of_week,
                mr.air_temperature,
                b.primary_use,
                b.industry
            FROM meter_readings mr
            JOIN energy_meters em ON mr.meter_id = em.id
            JOIN buildings b ON em.building_id = b.id
            WHERE b.id = $1
            AND mr.timestamp >= CURRENT_DATE - INTERVAL '90 days'
        """, building_id)
        
        # Generate pattern embeddings
        pattern_text = self.create_pattern_description(patterns)
        embedding = self.embedding_model.encode([pattern_text])[0]
        
        # Store in Milvus
        await self.milvus.insert("building_energy_patterns", [{
            "building_id": building_id,
            "pattern_type": "consumption_profile",
            "pattern_embedding": embedding.tolist(),
            "confidence_score": 0.95,
            "timestamp": int(time.time() * 1000)
        }])
```

## 📈 Real Data Volume Planning

### BDG2 Scale Architecture Considerations
```yaml
Data Volume Projections:
  Hourly Readings per Building: 8,760 per year per meter
  Total Annual Data Points: ~26M for full BDG2 dataset
  Storage Requirements:
    - PostgreSQL: ~5GB for 2 years of readings + metadata
    - Milvus: ~2GB for pattern embeddings
    - Redis Cache: 512MB for hot data
    
Query Performance Targets:
  - Single building hourly data (1 year): <200ms
  - Multi-building comparison (10 buildings): <500ms  
  - Anomaly detection scan (daily): <2s
  - Pattern similarity search: <100ms

Memory Allocation for BDG2:
  - Database connections: 1GB
  - Pattern analysis: 2GB
  - Real-time processing: 1GB
  - LLM model overhead: 8GB
  - System overhead: 4GB
  Total: 16GB (Perfect fit for M1 MacBook Pro)
```

## 🔍 BDG2-Specific Agent Capabilities

### Enhanced Data Intelligence Agent
```python
class BDG2DataIntelligenceAgent:
    async def analyze_building_vs_peers(self, building_id: str):
        """Compare building performance against BDG2 peer group"""
        
        # Get building characteristics
        building = await self.get_building_profile(building_id)
        
        # Find similar buildings in BDG2 dataset
        similar_buildings = await self.milvus.search(
            collection_name="building_energy_patterns",
            search_params={
                "metric_type": "COSINE",
                "params": {"nprobe": 10}
            },
            query_embedding=building.pattern_embedding,
            limit=10
        )
        
        # Benchmark analysis
        benchmarks = await self.calculate_peer_benchmarks(
            building, similar_buildings
        )
        
        return EnergyBenchmarkReport(
            building_performance=building.efficiency_score,
            peer_average=benchmarks.average_efficiency,
            percentile_rank=benchmarks.percentile,
            improvement_potential=benchmarks.savings_opportunity
        )
        
    async def detect_gepiii_patterns(self, building_id: str):
        """Apply GEPIII competition learnings to building analysis"""
        
        # Retrieve GEPIII competition insights
        competition_insights = await self.milvus.search(
            collection_name="gepiii_insights",
            query_embedding=self.encode_building_features(building_id),
            limit=5
        )
        
        # Apply proven prediction models
        predictions = []
        for insight in competition_insights:
            model_prediction = await self.apply_competition_model(
                building_id, insight.prediction_model
            )
            predictions.append(model_prediction)
            
        return GEPIIIInsightReport(
            predictions=predictions,
            confidence_scores=[p.accuracy for p in predictions],
            recommended_model=max(predictions, key=lambda p: p.accuracy)
        )
```

This BDG2 integration provides the EAIO system with validated real-world data patterns, enabling more accurate AI training and benchmarking against actual energy consumption behaviors from 1,636 buildings across diverse use cases and geographic regions. 