# Database Performance Specifications
**EAIO Development Mode (T.*) - Database Layer Performance Requirements**

## 🎯 Overview

This document defines comprehensive performance specifications for the EAIO database layer, including PostgreSQL + TimescaleDB and Milvus vector database, aligned with BDG2 dataset scale and real-world energy management requirements.

---

## 📊 Performance Targets Summary

| Component | Metric | Target | Measurement | Business Justification |
|-----------|--------|--------|-------------|----------------------|
| **PostgreSQL Queries** | Response Time | p95 < 100ms | Query duration | Real-time dashboard responsiveness |
| **TimescaleDB Aggregations** | Aggregation Speed | p95 < 200ms | Time-series queries | Interactive energy analytics |
| **Milvus Vector Search** | Similarity Search | p95 < 50ms | Vector operations | Building pattern matching |
| **Connection Pool** | Concurrent Users | 200+ connections | PgBouncer metrics | Multi-tenant support |
| **Data Ingestion** | Throughput | 10K records/sec | ETL pipeline | Real-time meter data processing |
| **Database Size** | Storage Efficiency | 500GB max | Disk usage | Cost optimization |

---

## 🗄️ PostgreSQL Performance Specifications

### Query Performance Requirements

#### **SPEC-DB-001: Single Building Data Retrieval**
```sql
-- Test Query: Get building overview with latest readings
SELECT 
    b.building_id, b.building_name, b.floor_area,
    bt.type_name, cz.zone_name,
    COUNT(DISTINCT m.meter_id) as meter_count,
    MAX(ec.timestamp) as last_reading
FROM buildings b
JOIN building_types bt ON b.building_type_id = bt.id
JOIN climate_zones cz ON b.climate_zone_id = cz.id
LEFT JOIN meters m ON b.building_id = m.building_id
LEFT JOIN energy_consumption ec ON m.meter_id = ec.meter_id
WHERE b.building_id = $1
GROUP BY b.building_id, b.building_name, b.floor_area, bt.type_name, cz.zone_name;
```

**Performance Requirements**:
- **Response Time**: p95 < 10ms, p99 < 20ms
- **Execution Plan**: Index seek only, no table scans
- **Memory Usage**: < 1MB per query
- **CPU Usage**: < 0.1% CPU per query

**Test Scenarios**:
- Single building lookup (most common)
- Non-existent building ID (error handling)
- Building with maximum meter count
- Building with historical data gaps

---

#### **SPEC-DB-002: Time-Series Energy Data Queries**
```sql
-- Test Query: Hourly energy consumption for last 30 days
SELECT 
    DATE_TRUNC('hour', timestamp) as hour,
    SUM(energy_value) as total_consumption,
    AVG(energy_value) as avg_consumption,
    COUNT(*) as reading_count
FROM energy_consumption 
WHERE building_id = $1 
    AND timestamp >= NOW() - INTERVAL '30 days'
    AND timestamp < NOW()
GROUP BY DATE_TRUNC('hour', timestamp)
ORDER BY hour;
```

**Performance Requirements**:
- **Response Time**: p95 < 50ms, p99 < 100ms
- **Data Volume**: 720 hours × multiple meters
- **Index Usage**: TimescaleDB chunk exclusion
- **Memory Usage**: < 10MB per query

**Test Scenarios**:
- 30-day historical data (standard dashboard)
- 1-year historical data (analytics view)
- Multiple buildings comparison
- Peak usage period analysis

---

#### **SPEC-DB-003: Building Portfolio Aggregations**
```sql
-- Test Query: Portfolio-wide energy performance
SELECT 
    bt.type_name,
    COUNT(DISTINCT b.building_id) as building_count,
    SUM(daily_stats.total_consumption) as total_energy,
    AVG(daily_stats.total_consumption / b.floor_area) as energy_intensity
FROM buildings b
JOIN building_types bt ON b.building_type_id = bt.id
JOIN (
    SELECT 
        building_id,
        DATE(timestamp) as date,
        SUM(energy_value) as total_consumption
    FROM energy_consumption 
    WHERE timestamp >= CURRENT_DATE - INTERVAL '7 days'
    GROUP BY building_id, DATE(timestamp)
) daily_stats ON b.building_id = daily_stats.building_id
GROUP BY bt.type_name
ORDER BY total_energy DESC;
```

**Performance Requirements**:
- **Response Time**: p95 < 200ms, p99 < 500ms
- **Data Volume**: 1,636+ buildings, 7 days of data
- **Parallel Processing**: Use of parallel query execution
- **Memory Usage**: < 100MB per query

---

### Connection Pool Performance

#### **SPEC-DB-004: PgBouncer Connection Management**
```yaml
connection_pool_specs:
  max_client_connections: 200
  max_server_connections: 50
  pool_mode: transaction
  default_pool_size: 25
  reserve_pool_size: 5
  
performance_targets:
  connection_acquisition: p95_less_than_1ms
  pool_utilization: optimal_70_80_percent
  connection_lifetime: 1_hour_max
  idle_timeout: 5_minutes
```

**Load Testing Requirements**:
- 200 concurrent users for 30 minutes
- Mixed read/write workload
- Connection churn simulation
- Failover scenario testing

---

## 🔍 Milvus Vector Database Specifications

### Vector Similarity Search Performance

#### **SPEC-VDB-001: Building Characteristics Similarity**
```python
# Test Operation: Find similar buildings by characteristics
search_params = {
    "collection_name": "building_characteristics",
    "query_vectors": [building_embedding_384d],
    "search_params": {
        "metric_type": "COSINE",
        "params": {"ef": 100}
    },
    "limit": 10,
    "output_fields": ["building_id", "building_type", "floor_area"]
}
```

**Performance Requirements**:
- **Search Latency**: p95 < 50ms, p99 < 100ms
- **Recall Accuracy**: recall@10 > 95%
- **Throughput**: 1,000 searches/second
- **Memory Usage**: < 2GB for index

**Test Scenarios**:
- Single vector search (user query)
- Batch vector search (10 vectors)
- Cold start performance (first search)
- Concurrent search load

---

#### **SPEC-VDB-002: Energy Pattern Similarity**
```python
# Test Operation: Find buildings with similar energy patterns
search_params = {
    "collection_name": "energy_patterns",
    "query_vectors": [pattern_embedding_384d],
    "search_params": {
        "metric_type": "COSINE", 
        "params": {"ef": 100}
    },
    "limit": 20,
    "expr": "season == 'winter'",  # Filtered search
    "output_fields": ["building_id", "pattern_type", "season"]
}
```

**Performance Requirements**:
- **Search Latency**: p95 < 75ms, p99 < 150ms
- **Filtered Search**: Additional 25ms overhead max
- **Index Build Time**: < 2 hours for 100K patterns
- **Storage Efficiency**: < 1GB per 100K vectors

---

### Data Synchronization Performance

#### **SPEC-VDB-003: PostgreSQL-Milvus Sync**
```yaml
sync_performance_specs:
  real_time_sync:
    latency: p95_less_than_5_minutes
    throughput: 1000_updates_per_minute
    batch_size: 100_records_optimal
    
  bulk_sync:
    throughput: 10k_records_per_minute
    memory_usage: less_than_4gb
    cpu_usage: less_than_80_percent
    
  conflict_resolution:
    duplicate_detection: 100_percent_accuracy
    resolution_time: less_than_1_second
```

---

## 📈 Performance Testing Framework

### Load Testing Specifications

#### **SPEC-PERF-001: Database Load Testing**
```yaml
load_test_scenarios:
  baseline_load:
    concurrent_users: 50
    duration: 30_minutes
    query_mix:
      - single_building: 40_percent
      - time_series: 35_percent  
      - aggregations: 20_percent
      - admin_queries: 5_percent
      
  peak_load:
    concurrent_users: 200
    duration: 15_minutes
    query_intensity: 2x_baseline
    
  stress_test:
    concurrent_users: 500
    duration: 5_minutes
    target: identify_breaking_point
    
  endurance_test:
    concurrent_users: 100
    duration: 8_hours
    target: memory_leak_detection
```

#### **SPEC-PERF-002: BDG2 Data Scale Testing**
```yaml
data_scale_tests:
  bdg2_full_dataset:
    buildings: 1636
    meters: 3053
    data_points: 53.6_million
    time_range: 2016_2017
    
  performance_validation:
    query_response: maintain_targets_at_scale
    indexing_time: complete_within_4_hours
    storage_usage: under_500gb_total
    backup_time: complete_within_2_hours
```

### Monitoring and Alerting

#### **SPEC-MON-001: Performance Monitoring**
```yaml
monitoring_requirements:
  real_time_metrics:
    - query_response_time_p95_p99
    - connection_pool_utilization
    - vector_search_latency
    - database_cpu_memory_usage
    - disk_io_operations_per_second
    
  alert_thresholds:
    query_latency: p95_exceeds_target_by_50_percent
    connection_pool: utilization_above_90_percent
    disk_space: usage_above_80_percent
    error_rate: above_1_percent_for_5_minutes
    
  dashboard_requirements:
    - real_time_performance_overview
    - historical_trend_analysis
    - query_pattern_analysis
    - resource_utilization_tracking
```

---

## 🧪 Test Implementation Examples

### PostgreSQL Performance Test
```python
import asyncio
import asyncpg
import time
from statistics import quantile

async def test_single_building_query_performance():
    """Test SPEC-DB-001: Single Building Data Retrieval"""
    
    # Setup connection pool
    pool = await asyncpg.create_pool(
        "postgresql://user:pass@localhost:5432/eaio",
        min_size=10, max_size=20
    )
    
    building_ids = ["BLD_001", "BLD_002", "BLD_003"]  # Test building IDs
    response_times = []
    
    # Run 1000 queries to get statistical significance
    for _ in range(1000):
        building_id = random.choice(building_ids)
        
        start_time = time.perf_counter()
        async with pool.acquire() as conn:
            result = await conn.fetchrow("""
                SELECT b.building_id, b.building_name, 
                       COUNT(DISTINCT m.meter_id) as meter_count
                FROM buildings b
                LEFT JOIN meters m ON b.building_id = m.building_id
                WHERE b.building_id = $1
                GROUP BY b.building_id, b.building_name
            """, building_id)
        end_time = time.perf_counter()
        
        response_times.append((end_time - start_time) * 1000)  # Convert to ms
    
    # Validate performance targets
    p95_latency = quantile(response_times, 0.95)
    p99_latency = quantile(response_times, 0.99)
    
    assert p95_latency < 10.0, f"P95 latency {p95_latency}ms exceeds 10ms target"
    assert p99_latency < 20.0, f"P99 latency {p99_latency}ms exceeds 20ms target"
    
    await pool.close()
    return {
        "avg_latency": sum(response_times) / len(response_times),
        "p95_latency": p95_latency,
        "p99_latency": p99_latency
    }
```

### Milvus Performance Test
```python
from pymilvus import Collection, connections
import numpy as np
import time

def test_vector_search_performance():
    """Test SPEC-VDB-001: Building Characteristics Similarity"""
    
    # Connect to Milvus
    connections.connect("default", host="localhost", port="19530")
    collection = Collection("building_characteristics")
    
    # Generate test vectors (384 dimensions)
    test_vectors = np.random.random((100, 384)).astype(np.float32)
    search_latencies = []
    
    # Warm up
    collection.search(
        data=test_vectors[:1],
        anns_field="embedding",
        param={"metric_type": "COSINE", "params": {"ef": 100}},
        limit=10
    )
    
    # Performance test
    for vector in test_vectors:
        start_time = time.perf_counter()
        results = collection.search(
            data=[vector],
            anns_field="embedding", 
            param={"metric_type": "COSINE", "params": {"ef": 100}},
            limit=10,
            output_fields=["building_id", "building_type"]
        )
        end_time = time.perf_counter()
        
        search_latencies.append((end_time - start_time) * 1000)
    
    # Validate performance
    p95_latency = np.percentile(search_latencies, 95)
    p99_latency = np.percentile(search_latencies, 99)
    
    assert p95_latency < 50.0, f"P95 search latency {p95_latency}ms exceeds 50ms"
    assert p99_latency < 100.0, f"P99 search latency {p99_latency}ms exceeds 100ms"
    
    return {
        "avg_latency": np.mean(search_latencies),
        "p95_latency": p95_latency,
        "p99_latency": p99_latency,
        "total_searches": len(search_latencies)
    }
```

---

## ✅ Acceptance Criteria

### Performance Validation Checklist
- [ ] All PostgreSQL queries meet response time targets
- [ ] TimescaleDB hypertables perform within specifications
- [ ] Milvus vector searches achieve latency targets
- [ ] Connection pooling handles target concurrent load
- [ ] BDG2 full dataset ingestion completes within time limits
- [ ] System maintains performance under sustained load
- [ ] Monitoring and alerting systems operational
- [ ] Performance degradation detection functional

### Quality Gates
- [ ] Load testing passes for all scenarios
- [ ] Performance regression tests included in CI/CD
- [ ] Monitoring dashboards created and validated
- [ ] Documentation updated with performance characteristics
- [ ] Performance tuning recommendations documented

This comprehensive performance specification ensures the EAIO database layer meets enterprise-grade requirements for real-time building energy management at scale. 