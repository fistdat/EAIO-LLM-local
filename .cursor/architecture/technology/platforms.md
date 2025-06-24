# EAIO Technology Platform Architecture
**Architecture Mode (A.*) - Hybrid LLM Technology Stack with BDG2 Integration**

## 🎯 Executive Summary

The EAIO technology platform supports **hybrid LLM deployment** combining **local privacy-first models** with **external API services** (ChatGPT, DeepSeek, Gemini), optimized for **MacBook Pro M1 hardware** while providing enterprise-scale energy management capabilities with flexible AI service selection.

## 🧠 Hybrid LLM Infrastructure

### LLM Service Architecture Strategy
```yaml
Hybrid LLM Deployment:
  Primary Strategy: Local-first with API fallback
  Privacy Levels: 
    - PRIVATE: Local LLM only (sensitive building data)
    - ENHANCED: External API allowed (aggregated insights)
    - PUBLIC: External API preferred (general analysis)
  
Cost Optimization:
  - Local LLM: Zero per-token cost, hardware investment
  - External APIs: Pay-per-use, premium capabilities
  - Intelligent routing based on query complexity and privacy
```

### 🏠 Local LLM Stack (Primary)

#### Model Selection Matrix
| Model | Size | RAM Usage | Tokens/sec | Use Case | Privacy Level |
|-------|------|-----------|------------|----------|---------------|
| **Llama-3.2-3B-Instruct** | 3B | 3GB | 45-60 | Real-time ops, private data | HIGH |
| **Qwen2.5-7B-Instruct** | 7B | 7GB | 25-35 | Complex analysis, planning | HIGH |
| **DeepSeek-Coder-6.7B** | 6.7B | 6GB | 30-40 | Code generation, debugging | HIGH |
| **Phi-3-Mini-4K** | 3.8B | 4GB | 50-65 | Fast responses, simple tasks | HIGH |

#### Local Infrastructure
```yaml
Primary Inference Engine: Ollama v0.3+
  Advantages:
    - Native M1 optimization
    - GGUF format support
    - Built-in model management
    - OpenAI-compatible API
    - Zero external data sharing
    
Performance Characteristics:
  - Cold start: 5-15 seconds
  - Warm inference: 20-100 tokens/sec
  - Context window: 4K-32K tokens
  - Concurrent requests: 1-2 (M1 16GB limitation)
    
Privacy Benefits:
  - 100% local processing
  - No data transmission
  - Complete control over model versions
  - Regulatory compliance (GDPR, CCPA)
```

### 🌐 External LLM API Integration

#### Supported API Providers
```yaml
Tier 1 Providers (Production Ready):
  OpenAI:
    Models: 
      - gpt-4o: Advanced reasoning, multimodal
      - gpt-4o-mini: Cost-effective, fast
      - gpt-3.5-turbo: High-speed, affordable
    API: REST + Streaming
    Cost: $0.50-$60 per 1M tokens
    Use Cases: Complex analysis, creative solutions
    
  DeepSeek API:
    Models:
      - deepseek-chat: General purpose, cost-effective
      - deepseek-coder: Specialized code generation
      - deepseek-reasoning: Advanced logical reasoning
    API: OpenAI-compatible REST
    Cost: $0.14-$2.00 per 1M tokens
    Use Cases: Technical analysis, energy algorithms
    
  Google Gemini:
    Models:
      - gemini-1.5-pro: Advanced reasoning, large context
      - gemini-1.5-flash: Fast responses, multimodal
      - gemini-2.0-flash-exp: Latest experimental features
    API: REST with Vertex AI integration
    Cost: $1.25-$7.00 per 1M tokens
    Use Cases: Data analysis, pattern recognition

Tier 2 Providers (Future Integration):
  Anthropic Claude: 
    - claude-3.5-sonnet: Excellent for technical writing
    - claude-3-haiku: Fast, cost-effective
  
  Mistral AI:
    - mistral-large: European data compliance
    - mistral-medium: Balanced performance/cost
```

#### API Integration Architecture
```python
# Hybrid LLM Service Router
class HybridLLMService:
    def __init__(self):
        self.local_ollama = OllamaClient("http://localhost:11434")
        self.openai_client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
        self.deepseek_client = OpenAI(
            api_key=os.getenv("DEEPSEEK_API_KEY"),
            base_url="https://api.deepseek.com"
        )
        self.gemini_client = genai.GenerativeModel('gemini-1.5-pro')
        
    async def route_request(self, query: LLMRequest) -> LLMResponse:
        """Intelligent routing based on privacy, complexity, and cost"""
        
        # Privacy-first routing
        if query.privacy_level == "PRIVATE":
            return await self.local_ollama.generate(query)
            
        # Complexity-based routing
        if query.complexity_score > 0.8:
            if query.domain == "technical":
                return await self.deepseek_client.generate(query)
            elif query.domain == "analysis":
                return await self.gemini_client.generate(query)
            else:
                return await self.openai_client.generate(query)
                
        # Cost-optimized routing
        if query.cost_sensitivity == "low":
            return await self.local_ollama.generate(query)
        else:
            return await self.select_cheapest_api(query)
            
    async def fallback_strategy(self, query: LLMRequest, failed_service: str):
        """Fallback when primary service fails"""
        if failed_service == "local":
            return await self.deepseek_client.generate(query)  # Most cost-effective
        else:
            return await self.local_ollama.generate(query)  # Always available
```

### 🔄 LLM Service Selection Strategy

#### Decision Matrix
```yaml
Query Classification:
  Building Data Analysis:
    Privacy: HIGH → Local LLM only
    Complexity: MEDIUM → Qwen2.5-7B local
    Fallback: DeepSeek API for complex algorithms
    
  Energy Optimization Planning:
    Privacy: MEDIUM → Local preferred, API allowed
    Complexity: HIGH → GPT-4o or Gemini-1.5-pro
    Cost Consideration: DeepSeek for technical calculations
    
  General User Queries:
    Privacy: LOW → API preferred for best experience
    Response Speed: HIGH → GPT-3.5-turbo or local Phi-3
    
  Code Generation:
    Privacy: MEDIUM → DeepSeek-Coder local or API
    Complexity: Variable → Route based on code complexity
    
Performance Requirements:
  Real-time Monitoring: Local LLM only (<200ms)
  Strategic Planning: External API allowed (<5s)
  Batch Analysis: External API preferred (cost-effective)
  Emergency Response: Local LLM only (reliability)
```

### 🔐 Security & Privacy Framework

#### Data Classification & Routing
```yaml
Data Sensitivity Levels:
  CRITICAL (Local Only):
    - Individual building energy consumption
    - Specific equipment performance data
    - Proprietary optimization algorithms
    - Real-time operational status
    
  SENSITIVE (Local Preferred):
    - Aggregated building performance
    - Anonymous pattern analysis
    - General optimization strategies
    - Benchmarking comparisons
    
  PUBLIC (API Allowed):
    - Industry best practices research
    - General energy efficiency concepts
    - Weather correlation analysis
    - Academic research insights

Privacy Protection Measures:
  - Automatic data anonymization for API calls
  - Configurable privacy levels per building/client
  - Audit logging for all external API usage
  - Zero-trust principle for sensitive data
```

#### API Security Implementation
```python
# Secure API Integration
class SecureAPIClient:
    def __init__(self, provider: str):
        self.provider = provider
        self.encryption = Fernet(os.getenv(f"{provider}_ENCRYPTION_KEY"))
        self.rate_limiter = AsyncLimiter(max_rate=100, time_period=60)
        
    async def secure_call(self, request: LLMRequest) -> LLMResponse:
        # Data sanitization
        sanitized_request = self.sanitize_sensitive_data(request)
        
        # Rate limiting
        async with self.rate_limiter:
            # Encrypted transmission
            encrypted_payload = self.encryption.encrypt(
                json.dumps(sanitized_request.dict()).encode()
            )
            
            # API call with retry logic
            response = await self.call_with_retry(encrypted_payload)
            
            # Response validation
            validated_response = self.validate_response(response)
            
            # Audit logging
            await self.log_api_usage(request.id, self.provider, response.tokens_used)
            
            return validated_response
```

## 💰 Cost Optimization Strategy

### Hybrid Cost Model
```yaml
Local LLM Costs:
  Hardware Investment: $2,000-4,000 (M1 MacBook Pro)
  Electricity: ~$0.001 per query (negligible)
  Maintenance: Software updates only
  Scaling: Linear hardware investment
  
External API Costs (Per Month Estimates):
  Light Usage (1K queries): $10-50
  Medium Usage (10K queries): $100-500  
  Heavy Usage (100K queries): $1,000-5,000
  Enterprise Usage (1M queries): $10,000-50,000
  
Cost Optimization Rules:
  - Simple queries → Local LLM (free)
  - Complex analysis → DeepSeek API (cheapest)
  - Creative tasks → OpenAI (best quality)
  - Technical coding → DeepSeek Coder (specialized)
  - Multimodal needs → Gemini (vision capabilities)
```

### Intelligent Cost Management
```python
class CostOptimizer:
    def __init__(self):
        self.monthly_budget = float(os.getenv("LLM_MONTHLY_BUDGET", "1000"))
        self.cost_tracker = {}
        self.provider_costs = {
            "local": 0.0,
            "openai": {"gpt-4o": 0.0025, "gpt-3.5-turbo": 0.0005},
            "deepseek": {"deepseek-chat": 0.00014, "deepseek-coder": 0.00014},
            "gemini": {"gemini-1.5-pro": 0.00125, "gemini-1.5-flash": 0.00035}
        }
        
    async def select_cost_optimal_provider(self, query: LLMRequest) -> str:
        current_spend = sum(self.cost_tracker.values())
        remaining_budget = self.monthly_budget - current_spend
        
        # Budget emergency - use local only
        if remaining_budget < self.monthly_budget * 0.1:
            return "local"
            
        # Calculate cost-benefit for each provider
        options = self.calculate_cost_benefit(query)
        
        # Select best option within budget
        for provider, score in sorted(options.items(), key=lambda x: x[1], reverse=True):
            estimated_cost = self.estimate_query_cost(query, provider)
            if estimated_cost <= remaining_budget * 0.05:  # Max 5% of remaining budget per query
                return provider
                
        return "local"  # Fallback to free option
```

## 📊 Performance Monitoring & Analytics

### LLM Performance Metrics
```yaml
Local LLM Monitoring:
  - Response time: <500ms target
  - Token throughput: 20-60 tokens/sec
  - Memory usage: <8GB sustained
  - GPU utilization: <90% sustained
  - Error rate: <1%
  
External API Monitoring:
  - Response time: <3s target
  - API availability: >99.9%
  - Cost per query: Track against budget
  - Rate limit utilization: <80%
  - Quality scores: User feedback based
  
System Health Checks:
  - Local model availability
  - API key validity and quotas
  - Network connectivity to API providers
  - Failover system responsiveness
```

### Hybrid Performance Dashboard
```python
# Performance Analytics
class LLMPerformanceMonitor:
    def __init__(self):
        self.metrics_store = TimescaleDB("llm_performance")
        
    async def track_request(self, request: LLMRequest, response: LLMResponse):
        await self.metrics_store.insert({
            "timestamp": datetime.utcnow(),
            "provider": response.provider,
            "model": response.model,
            "response_time_ms": response.response_time,
            "tokens_used": response.tokens_used,
            "cost_usd": response.cost,
            "quality_score": response.quality_score,
            "privacy_level": request.privacy_level,
            "query_complexity": request.complexity_score
        })
        
    async def generate_optimization_report(self) -> Dict:
        """Generate recommendations for LLM usage optimization"""
        
        metrics = await self.metrics_store.query("""
            SELECT 
                provider,
                AVG(response_time_ms) as avg_response_time,
                SUM(cost_usd) as total_cost,
                AVG(quality_score) as avg_quality,
                COUNT(*) as request_count
            FROM llm_performance 
            WHERE timestamp >= NOW() - INTERVAL '30 days'
            GROUP BY provider
        """)
        
        return {
            "cost_optimization": self.analyze_cost_efficiency(metrics),
            "performance_optimization": self.analyze_response_times(metrics),
            "quality_optimization": self.analyze_quality_scores(metrics),
            "recommendations": self.generate_recommendations(metrics)
        }
```

## 🔧 Configuration Management

### Environment Configuration
```yaml
# .env configuration for hybrid LLM setup
LLM_STRATEGY=hybrid  # local, hybrid, api-only

# Local LLM Configuration
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_DEFAULT_MODEL=llama3.2:3b-instruct-q4_K_M
OLLAMA_KEEP_ALIVE=30m

# External API Configuration
OPENAI_API_KEY=sk-...
OPENAI_ORG_ID=org-...
DEEPSEEK_API_KEY=sk-...
GOOGLE_API_KEY=AIza...

# Cost Controls
LLM_MONTHLY_BUDGET=1000.0
COST_ALERT_THRESHOLD=0.8
EMERGENCY_LOCAL_ONLY=false

# Privacy Controls
DEFAULT_PRIVACY_LEVEL=PRIVATE
ALLOW_API_FOR_AGGREGATED=true
AUDIT_ALL_API_CALLS=true

# Performance Controls
LOCAL_TIMEOUT_SEC=10
API_TIMEOUT_SEC=30
MAX_CONCURRENT_LOCAL=2
MAX_CONCURRENT_API=10
```

### Dynamic Configuration
```python
# Runtime LLM configuration
class LLMConfig:
    def __init__(self):
        self.load_config()
        
    def load_config(self):
        self.strategy = os.getenv("LLM_STRATEGY", "hybrid")
        self.monthly_budget = float(os.getenv("LLM_MONTHLY_BUDGET", "1000"))
        self.default_privacy = os.getenv("DEFAULT_PRIVACY_LEVEL", "PRIVATE")
        
        # Provider availability check
        self.available_providers = self.check_provider_availability()
        
    async def check_provider_availability(self) -> Dict[str, bool]:
        """Check which LLM providers are currently available"""
        availability = {}
        
        # Local Ollama check
        try:
            response = await self.ollama_client.list_models()
            availability["local"] = len(response) > 0
        except:
            availability["local"] = False
            
        # API providers check
        for provider in ["openai", "deepseek", "gemini"]:
            availability[provider] = await self.check_api_availability(provider)
            
        return availability
        
    def get_optimal_provider(self, query: LLMRequest) -> str:
        """Get the optimal provider for this specific query"""
        
        # Privacy constraints
        if query.privacy_level == "PRIVATE":
            return "local" if self.available_providers["local"] else None
            
        # Performance requirements  
        if query.urgency == "HIGH":
            if self.available_providers["local"]:
                return "local"
            return "openai"  # Fastest API
            
        # Cost optimization
        if query.cost_sensitivity == "HIGH":
            if self.available_providers["local"]:
                return "local"
            return "deepseek"  # Cheapest API
            
        # Quality requirements
        if query.complexity_score > 0.8:
            return "openai"  # Best quality for complex tasks
            
        return "local"  # Default to local when possible
```

This hybrid LLM architecture provides maximum flexibility while maintaining privacy, cost control, and performance optimization for the EAIO energy management system. 