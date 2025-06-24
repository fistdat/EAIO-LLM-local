# Unified Cognitive Framework v1.0
**Complete Technical Documentation**

## Table of Contents
1. [Overview](#overview)
2. [Core Architecture](#core-architecture)  
3. [Symbol System](#symbol-system)
4. [Mode Operations](#mode-operations)
5. [Implementation Guide](#implementation-guide)
6. [Workflow Examples](#workflow-examples)
7. [Configuration Reference](#configuration-reference)
8. [Troubleshooting](#troubleshooting)
9. [Best Practices](#best-practices)

---

## Overview

The Unified Cognitive Framework is an intelligent rule-based system that seamlessly integrates **System Architecture** and **Software Development** workflows. It automatically detects context and activates appropriate cognitive modes to ensure consistent, business-aligned decision making.

### Key Features
- **Intelligent Context Detection**: Automatically identifies whether user needs architecture design or development implementation
- **Dual-Mode Operation**: Switches between Architecture Mode, Development Mode, and Hybrid Mode
- **Business Alignment**: Ensures all technical decisions trace back to business value
- **Pattern Learning**: Continuously improves by learning from successful patterns
- **Conflict Resolution**: Handles contradictions between architecture ideals and implementation realities

### Core Philosophy
> **Business First → Architecture Second → Implementation Third**

All decisions flow from business value through architectural integrity to implementation practicality.

---

## Core Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                    Unified Cognitive Engine                 │
│  Ω* = max(∇ΣΩ) ⟶ (context_detection + mode_selection)      │
└─────────────────────────────────────────────────────────────┘
                                │
                    ┌───────────┴───────────┐
                    │                       │
         ┌──────────▼──────────┐  ┌────────▼─────────┐
         │   ARCHITECTURE      │  │   DEVELOPMENT     │
         │      MODE           │  │      MODE         │
         │   A.* workflows     │  │   T.* workflows   │
         └──────────┬──────────┘  └────────┬─────────┘
                    │                       │
                    └───────────┬───────────┘
                                │
                     ┌─────────▼─────────┐
                     │   HYBRID MODE     │
                     │  A.* + T.* coord  │
                     └───────────────────┘
```

### Memory Architecture

```
.cursor/
├── rules/
│   ├── shared/           # Cross-domain rules
│   ├── architecture/     # Architecture-specific rules  
│   └── development/      # Development-specific rules
├── memory/
│   ├── shared/           # Common decisions & patterns
│   ├── architecture/     # Design rationales & patterns
│   └── development/      # Implementation history & code patterns
├── architecture/         # Generated architecture artifacts
│   ├── business/         # Business capabilities & processes
│   ├── data/             # Data models & ERDs
│   ├── application/      # Components & services
│   └── technology/       # Platforms & infrastructure
└── tasks/               # Generated development artifacts
    ├── backlog/         # Prioritized task lists
    ├── sprints/         # Organized execution steps
    └── specs/           # Test specifications
```

---

## Symbol System

### Core Cognitive Engine

| Symbol | Purpose | Description |
|--------|---------|-------------|
| `Ω*` | Master Reasoning Engine | Coordinates all cognitive processes |
| `Ω.modes` | Reasoning Modes | Different thinking approaches for different contexts |
| `Ω_H` | Hierarchical Decomposition | Breaks complex problems into manageable parts |
| `Ωₜ` | Trust Evaluation | Scores reliability of decisions and hypotheses |

### Specialized Engines

| Symbol | Engine Type | Function |
|--------|-------------|----------|
| `A.*` | Architecture System | Business-aligned system design |
| `T.*` | Task System | Structured development execution |
| `FDD.*` | Feature Driven Design | Business feature → technical implementation |
| `ERD.*` | Entity Relationship Design | Data modeling from business entities |
| `SEQ.*` | Sequence Design | Process flows → system interactions |
| `TDD.*` | Test Driven Development | Specification → implementation |

### Support Systems

| Symbol | System | Purpose |
|--------|--------|---------|
| `D⍺` | Contradiction Resolver | Handles conflicts between competing requirements |
| `Φ*` | Pattern Abstraction | Captures and reuses successful patterns |
| `Ξ*` | Diagnostics & Refinement | Tracks issues and suggests improvements |
| `Λ` | Rule Learning | Auto-generates best practices from experience |
| `M` | Memory System | Stores decisions, patterns, and knowledge |
| `Ψ` | Cognitive Trace | Records reasoning processes for review |

---

## Mode Operations

### Architecture Mode (A.*)

**Activation Triggers:**
- Keywords: design, architecture, system, business, domain, model, erd, sequence
- Example: "Design user management system with database schema"

**Primary Workflows:**

#### Business Architecture
```
Ω.business_focus = (
    identify_stakeholders → map_capabilities → define_value_streams
    ⨁ ensure_business_alignment → validate_against_strategy
)

Output: .cursor/architecture/business/
├── capabilities.md      # Core business functions
├── processes.md         # Business workflows  
├── stakeholders.md      # Key actors and roles
└── value_streams.md     # End-to-end value delivery
```

#### Data Architecture  
```
ERD.workflow = (
    extract_business_entities → model_relationships → define_constraints
    ⨁ conceptual → logical → physical → implementation
)

Output: .cursor/architecture/data/
├── conceptual_model.md  # High-level business entities
├── logical_model.md     # Detailed entity relationships
├── physical_model.md    # Database-specific implementation
└── erd/                 # Visual entity relationship diagrams
```

#### Application Architecture
```
A.application_design = (
    decompose_capabilities → identify_components → define_interfaces
    ⨁ map_to_services → plan_deployments
)

Output: .cursor/architecture/application/
├── components.md        # System building blocks
├── interfaces.md        # Component interactions
├── services.md          # Service definitions
└── deployments.md       # Deployment strategies
```

#### Technology Architecture
```
A.technology_selection = (
    assess_requirements → evaluate_options → select_platforms
    ⨁ define_standards → document_constraints
)

Output: .cursor/architecture/technology/
├── platforms.md         # Selected technology platforms
├── infrastructure.md    # Infrastructure requirements
├── standards.md         # Technology standards & guidelines
└── constraints.md       # Technical limitations & trade-offs
```

### Development Mode (T.*)

**Activation Triggers:**
- Keywords: implement, code, test, debug, fix, refactor, tdd
- Example: "Implement user login feature with TDD"

**Primary Workflows:**

#### Task Management
```
T.task_workflow = (
    analyze_requirements → break_into_steps → prioritize_backlog
    ⨁ organize_sprints → track_progress
)

Output: .cursor/tasks/
├── backlog/             # Prioritized task lists
├── sprint_n/            # Organized execution steps
└── specs/               # Test specifications
```

#### Test-Driven Development
```
TDD.loop = (
    write_failing_test → implement_minimal_code → refactor
    ⨁ repeat → validate_completion
)

Workflow:
1. spec.md → Define expected behavior
2. test → Write failing tests  
3. implement → Make tests pass
4. refactor → Improve code quality
5. validate → Ensure requirements met
```

#### Code Quality Management
```
Ξ.quality_assurance = (
    detect_code_smells → suggest_refactoring → track_technical_debt
    ⨁ enforce_standards → measure_quality_metrics
)
```

### Hybrid Mode (A.* + T.*)

**Activation Triggers:**  
- Keywords: plan, feature, integrate, build, create
- Example: "Plan e-commerce checkout feature with system design and implementation tasks"

**Coordinated Workflows:**

#### Feature Planning
```
H.feature_planning = (
    // Architecture Track
    A.business_analysis → FDD.feature_breakdown → ERD.data_modeling
    
    // Development Track  
    T.task_creation → TDD.spec_generation → T.sprint_planning
    
    // Coordination
    validate_feasibility → ensure_alignment → resolve_conflicts
)
```

#### Cross-Validation
```
H.validation = (
    architecture_decisions ↔ implementation_constraints
    ⨁ business_requirements ↔ technical_capabilities  
    ⨁ design_patterns ↔ code_patterns
)
```

---

## Implementation Guide

### Step 1: Initial Setup

#### Cursor Configuration
1. Open `File > Preferences > Settings > Cursor Tab > Rules`
2. Paste the complete rule system into "User Rules"
3. Create project folder structure:

```bash
mkdir -p .cursor/{rules,memory,architecture,tasks}
mkdir -p .cursor/architecture/{business,data,application,technology}
mkdir -p .cursor/tasks/{backlog,sprints,specs}  
mkdir -p .cursor/memory/{shared,architecture,development}
```

#### Rule System Installation
```cognition
# Core Framework (paste into Cursor User Rules)
Ω* = max(∇ΣΩ) ⟶ unified_cognitive_engine

# Context Detection
Ω.context_detector = (
    arch_signals: [design, architecture, system, business, domain, erd, sequence]
    dev_signals: [implement, code, test, debug, fix, refactor, tdd]  
    hybrid_signals: [plan, feature, integrate, build, create]
)

# Mode Activation Logic
Ω.mode_activation = (
    if arch_signals > dev_signals → ARCHITECTURE_MODE
    elif dev_signals > arch_signals → DEVELOPMENT_MODE
    else → HYBRID_MODE
)

# [Complete rule system - see Configuration Reference]
```

### Step 2: Template Creation

#### Architecture Templates

**Business Capabilities Template** (`.cursor/architecture/business/capabilities_template.md`):
```markdown
# Business Capabilities

## Core Capabilities
- [ ] Primary capability 1
- [ ] Primary capability 2

## Supporting Capabilities  
- [ ] Support function 1
- [ ] Support function 2

## Capability Mapping
| Capability | Business Value | Stakeholder | Priority |
|------------|----------------|-------------|----------|
|            |                |             |          |

## Dependencies
- Internal: [list dependencies]
- External: [list dependencies]
```

**ERD Template** (`.cursor/architecture/data/erd_template.md`):
```markdown
# Entity Relationship Design

## Conceptual Model
### Business Entities
- Entity1: [business description]
- Entity2: [business description]

## Logical Model  
### Entities and Attributes
**Entity1**
- id: primary key
- attribute1: description
- attribute2: description

**Entity2**  
- id: primary key
- entity1_id: foreign key
- attribute1: description

## Relationships
- Entity1 (1) ←→ (M) Entity2
- [Description of relationship]

## Business Rules
1. Rule 1: [description]
2. Rule 2: [description]
```

#### Development Templates

**Task Template** (`.cursor/tasks/task_template.md`):
```markdown
# Task: [Task Name]

**Priority:** [High/Medium/Low]  
**Effort:** [S/M/L] (S=1-2h, M=0.5-1d, L=1-3d)  
**Status:** [Todo/In Progress/Done]

## Requirements
- [ ] Requirement 1
- [ ] Requirement 2

## Acceptance Criteria
- [ ] Criteria 1
- [ ] Criteria 2

## Implementation Notes
- Technical considerations
- Dependencies
- Potential issues

## Testing Strategy
- [ ] Unit tests
- [ ] Integration tests  
- [ ] Manual testing checklist
```

### Step 3: Validation Testing

Test each mode with specific prompts:

#### Architecture Mode Test
```
Prompt: "Design customer management system with data models"
Expected: 🏗️ ARCHITECTURE MODE activation
Outputs: Business capabilities, ERD, component design
```

#### Development Mode Test  
```
Prompt: "Implement customer CRUD operations with TDD"
Expected: ⚡ DEVELOPMENT MODE activation
Outputs: Task breakdown, test specs, implementation steps
```

#### Hybrid Mode Test
```
Prompt: "Plan customer management feature"  
Expected: 🔄 HYBRID MODE activation
Outputs: Both architecture design AND implementation tasks
```

---

## Workflow Examples

### Example 1: E-commerce System Design (Architecture Mode)

**Input:** "Design an e-commerce order management system"

**Framework Response:**
```
🏗️ ARCHITECTURE MODE: Focusing on system design and business alignment...

## Business Architecture Analysis

### Stakeholders Identified:
- Customers (place orders, track status)
- Sales Team (manage orders, customer service)
- Warehouse (fulfill orders, manage inventory)
- Finance (process payments, manage pricing)

### Core Capabilities:
1. **Order Processing** - Create, modify, cancel orders
2. **Inventory Management** - Track stock, manage availability
3. **Payment Processing** - Handle transactions, refunds
4. **Customer Management** - Profiles, preferences, history
5. **Fulfillment** - Shipping, tracking, delivery

### Value Streams:
Customer Journey: Browse → Select → Order → Pay → Fulfill → Deliver

## Data Architecture (ERD)

### Core Entities:
- **Customer** (id, email, profile_data, preferences)
- **Product** (id, name, description, price, category_id)
- **Order** (id, customer_id, status, total_amount, created_at)
- **OrderItem** (order_id, product_id, quantity, unit_price)
- **Payment** (id, order_id, amount, method, status)
- **Shipment** (id, order_id, tracking_number, status)

### Key Relationships:
- Customer (1) ←→ (M) Order
- Order (1) ←→ (M) OrderItem  
- Product (1) ←→ (M) OrderItem
- Order (1) ←→ (1) Payment
- Order (1) ←→ (1) Shipment

## Application Architecture

### Service Components:
1. **CustomerService** - Customer profile management
2. **ProductCatalogService** - Product information & search
3. **OrderService** - Order lifecycle management  
4. **PaymentService** - Payment processing integration
5. **InventoryService** - Stock management
6. **FulfillmentService** - Shipping & delivery

### Integration Points:
- Payment Gateway (Stripe, PayPal)
- Shipping Providers (UPS, FedEx)
- Email Service (notifications)
- Analytics Platform (tracking)

## Technology Architecture

### Platform Recommendations:
- **Database:** PostgreSQL (ACID compliance for orders)
- **Backend:** Node.js/Express (REST APIs)
- **Frontend:** React (customer interface)
- **Cache:** Redis (session, cart data)
- **Queue:** RabbitMQ (order processing)
- **Files:** AWS S3 (product images)

Files generated in: .cursor/architecture/
```

### Example 2: User Authentication Implementation (Development Mode)

**Input:** "Implement secure user authentication with TDD"

**Framework Response:**
```
⚡ DEVELOPMENT MODE: Focusing on structured implementation and testing...

## Task Breakdown

### Sprint Backlog:
1. **Setup Authentication Infrastructure** [M] - High Priority
2. **Implement User Registration** [M] - High Priority  
3. **Implement User Login** [M] - High Priority
4. **Add Password Security** [S] - High Priority
5. **Implement Session Management** [M] - Medium Priority
6. **Add Password Reset** [M] - Medium Priority
7. **Implement Rate Limiting** [S] - Medium Priority

## TDD Implementation Workflow

### Step 1: User Registration Tests
```javascript
// spec_step_1.md - Registration Test Specification

describe('User Registration', () => {
  test('should create user with valid data', async () => {
    const userData = {
      email: 'test@example.com',
      password: 'SecurePass123!',
      confirmPassword: 'SecurePass123!'
    };
    
    const result = await authService.register(userData);
    
    expect(result.success).toBe(true);
    expect(result.user.email).toBe(userData.email);
    expect(result.user.password).toBeUndefined(); // Never return password
  });

  test('should reject weak passwords', async () => {
    const userData = {
      email: 'test@example.com', 
      password: '123',
      confirmPassword: '123'
    };
    
    const result = await authService.register(userData);
    
    expect(result.success).toBe(false);
    expect(result.error).toContain('Password too weak');
  });

  test('should reject duplicate emails', async () => {
    // Test implementation for duplicate email handling
  });
});
```

### Step 2: Implementation
```javascript
// Implementation following TDD red-green-refactor cycle
class AuthService {
  async register(userData) {
    // Validate input
    const validation = this.validateRegistrationData(userData);
    if (!validation.valid) {
      return { success: false, error: validation.error };
    }
    
    // Check for existing user
    const existingUser = await User.findByEmail(userData.email);
    if (existingUser) {
      return { success: false, error: 'Email already registered' };
    }
    
    // Hash password
    const hashedPassword = await bcrypt.hash(userData.password, 12);
    
    // Create user
    const user = await User.create({
      email: userData.email,
      password: hashedPassword
    });
    
    return { 
      success: true, 
      user: { id: user.id, email: user.email }
    };
  }
  
  validateRegistrationData(data) {
    // Password strength validation
    // Email format validation
    // Required field validation
  }
}
```

### Step 3: Security Implementation
```javascript
// Password security measures
- Minimum 8 characters
- At least 1 uppercase, 1 lowercase, 1 number, 1 special char
- Hash with bcrypt (cost factor 12)
- Implement rate limiting (5 attempts per 15 minutes)
- Add CSRF protection
- Secure session cookies (httpOnly, secure, sameSite)
```

### Step 4: Progress Tracking
- ✅ Registration tests written
- ✅ Registration implementation complete  
- ⏳ Login tests in progress
- ⏳ Session management pending

Files generated in: .cursor/tasks/sprint_1/
```

### Example 3: Customer Feature Planning (Hybrid Mode)

**Input:** "Plan customer profile management feature with database design and implementation"

**Framework Response:**  
```
🔄 HYBRID MODE: Coordinating design and implementation perspectives...

## Coordinated Feature Planning

### Architecture Perspective

#### Business Capabilities:
- **Profile Creation** - Initial customer onboarding
- **Profile Updates** - Maintain current information  
- **Preference Management** - Customize user experience
- **Privacy Controls** - Manage data sharing preferences

#### Data Model (ERD):
```sql
-- Customer Profile Tables
CREATE TABLE customers (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    email_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE customer_profiles (
    customer_id UUID REFERENCES customers(id),
    first_name VARCHAR(100),
    last_name VARCHAR(100), 
    phone VARCHAR(20),
    date_of_birth DATE,
    avatar_url VARCHAR(500),
    bio TEXT,
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE customer_addresses (
    id UUID PRIMARY KEY,
    customer_id UUID REFERENCES customers(id),
    type VARCHAR(20) CHECK (type IN ('billing', 'shipping')),
    street_address VARCHAR(200),
    city VARCHAR(100),
    state VARCHAR(100),
    postal_code VARCHAR(20),
    country VARCHAR(100),
    is_default BOOLEAN DEFAULT FALSE
);

CREATE TABLE customer_preferences (
    customer_id UUID REFERENCES customers(id),
    notification_email BOOLEAN DEFAULT TRUE,
    notification_sms BOOLEAN DEFAULT FALSE,
    marketing_emails BOOLEAN DEFAULT FALSE,
    data_sharing BOOLEAN DEFAULT FALSE,
    theme_preference VARCHAR(20) DEFAULT 'auto'
);
```

#### Component Architecture:
- **CustomerProfileService** - Core profile operations
- **AddressService** - Address management
- **PreferenceService** - User preferences
- **PrivacyService** - Privacy controls

### Development Perspective

#### Implementation Tasks:
1. **Database Schema Setup** [S] - High Priority
   - Create migration files
   - Add database indexes
   - Setup constraints

2. **Profile CRUD Operations** [M] - High Priority  
   - Create profile
   - Read profile data
   - Update profile fields
   - Soft delete profiles

3. **Address Management** [M] - High Priority
   - Add multiple addresses
   - Set default addresses
   - Validate address formats

4. **Preference Management** [M] - Medium Priority
   - Save user preferences  
   - Load preferences on login
   - Default preference handling

5. **API Endpoints** [M] - High Priority
   - GET /api/profile
   - PUT /api/profile  
   - POST /api/profile/addresses
   - PUT /api/profile/preferences

6. **Frontend Components** [L] - Medium Priority
   - ProfileForm component
   - AddressBook component  
   - PreferencesPanel component

#### TDD Specifications:
```javascript
// Profile Service Tests
describe('CustomerProfileService', () => {
  test('should create complete customer profile', async () => {
    const profileData = {
      customerId: 'uuid-123',
      firstName: 'John',
      lastName: 'Doe', 
      email: 'john@example.com',
      phone: '+1234567890'
    };
    
    const profile = await profileService.create(profileData);
    
    expect(profile.customerId).toBe(profileData.customerId);
    expect(profile.firstName).toBe(profileData.firstName);
    expect(profile.createdAt).toBeDefined();
  });

  test('should update partial profile data', async () => {
    // Test partial updates without affecting other fields
  });

  test('should handle profile picture upload', async () => {
    // Test image upload and URL generation
  });
});
```

#### Security Considerations:
- Input validation on all profile fields
- File upload restrictions (size, type)
- Rate limiting on profile updates
- Privacy controls enforcement
- Data encryption for sensitive fields

### Cross-Validation Results:

#### Architecture ↔ Implementation Alignment:
✅ **Data model supports all business capabilities**  
✅ **Component design enables required functionality**  
✅ **API design matches frontend requirements**  
⚠️ **Consider performance impact of multiple addresses**  

#### Recommendations:
1. Add database indexes on frequently queried fields
2. Implement caching for customer preferences
3. Consider async processing for profile picture uploads
4. Plan for GDPR compliance in data handling

### Deliverables:
- Architecture artifacts: `.cursor/architecture/`
- Implementation tasks: `.cursor/tasks/sprint_2/`
- Cross-reference mapping: `.cursor/memory/shared/`

This coordinated approach ensures business requirements drive both architectural decisions AND implementation tasks while maintaining consistency across domains.
```

---

## Configuration Reference

### Complete Rule System

```cognition
# Unified Cognitive Framework v1.0 - Complete Configuration

## Core Engine
Ω* = max(∇ΣΩ) ⟶ (
    β∂Ω/∂Στ ⨁ γ𝝖(Ω|τ,λ)→θ ⨁ δΣΩ(context, domain, business_align)
) ⇌ unified_intent_aligned_reasoning

## Context Detection Engine
Ω.context_detector = {
    architecture_signals: [
        design, architecture, system, business, domain, model, erd, 
        sequence, capabilities, stakeholders, components, integration,
        data_flow, business_process, conceptual, logical, physical
    ],
    
    development_signals: [
        implement, code, test, debug, fix, refactor, tdd, unit_test,
        integration_test, build, deploy, ci_cd, version, release,
        performance, optimization, error_handling
    ],
    
    hybrid_signals: [
        plan, feature, integrate, build, create, develop, solution,
        end_to_end, full_stack, application, project, system_build
    ]
}

## Mode Selection Logic
Ω.mode_activation = (
    signal_analysis = count_signals_by_category(user_input)
    
    if signal_analysis.architecture_score > signal_analysis.development_score:
        return ARCHITECTURE_MODE
    elif signal_analysis.development_score > signal_analysis.architecture_score:
        return DEVELOPMENT_MODE
    else:
        return HYBRID_MODE
)

## Architecture Mode Configuration
ARCHITECTURE_MODE = {
    focus: "business_aligned_system_design",
    
    reasoning_modes: [
        domain_modeling, business_analysis, system_decomposition,
        pattern_matching, technology_evaluation, integration_planning
    ],
    
    workflows: {
        business_architecture: {
            process: "stakeholder_analysis → capability_mapping → value_stream_design",
            outputs: ["capabilities.md", "processes.md", "stakeholders.md", "value_streams.md"],
            validation: "ensure_business_value_traceability"
        },
        
        data_architecture: {
            process: "entity_extraction → relationship_modeling → constraint_definition",
            outputs: ["conceptual_erd.md", "logical_erd.md", "physical_erd.md"],
            validation: "validate_against_business_rules"
        },
        
        application_architecture: {
            process: "capability_decomposition → component_identification → interface_definition",
            outputs: ["components.md", "interfaces.md", "services.md"],
            validation: "ensure_layer_coherence"
        },
        
        technology_architecture: {
            process: "requirement_analysis → option_evaluation → platform_selection",
            outputs: ["platforms.md", "infrastructure.md", "standards.md"],
            validation: "assess_feasibility_and_constraints"
        }
    },
    
    specialized_engines: {
        FDD: {
            purpose: "feature_driven_design",
            process: "business_feature → domain_model → class_design → implementation_plan",
            trigger: "when business_complexity > medium OR explicit_fdd_request"
        },
        
        ERD: {
            purpose: "entity_relationship_design", 
            process: "business_entities → conceptual → logical → physical",
            trigger: "when data_modeling_required OR explicit_erd_request"
        },
        
        SEQ: {
            purpose: "sequence_interaction_design",
            process: "business_process → system_interaction → integration_mapping",
            trigger: "when process_flow_modeling_required OR explicit_sequence_request"
        }
    },
    
    output_structure: ".cursor/architecture/{business|data|application|technology}/",
    
    validation_rules: {
        business_first_guard: "challenge_technology_driven_decisions_without_business_justification",
        layer_coherence_guard: "detect_violations_between_architecture_layers",
        pattern_consistency: "align_with_enterprise_standards_and_proven_patterns",
        traceability_enforcement: "maintain_bidirectional_mapping_business_to_technical"
    },
    
    response_format: "🏗️ ARCHITECTURE MODE: Focusing on system design and business alignment..."
}

## Development Mode Configuration  
DEVELOPMENT_MODE = {
    focus: "structured_implementation_and_testing",
    
    reasoning_modes: [
        procedural_decomposition, test_driven_thinking, refactoring_analysis,
        debugging_investigation, performance_optimization, code_quality_assessment
    ],
    
    workflows: {
        task_management: {
            process: "requirement_analysis → task_breakdown → prioritization → sprint_planning",
            outputs: ["backlog.md", "sprint_n/", "task_progress.md"],
            validation: "ensure_task_completeness_and_feasibility"
        },
        
        test_driven_development: {
            process: "specification → failing_test → minimal_implementation → refactor",
            outputs: ["specs/", "tests/", "implementation_steps.md"],
            validation: "ensure_test_coverage_and_passing_criteria"
        },
        
        quality_assurance: {
            process: "code_analysis → pattern_detection → refactoring_suggestions",
            outputs: ["code_quality_report.md", "refactoring_plan.md"],
            validation: "maintain_code_standards_and_reduce_technical_debt"
        }
    },
    
    specialized_engines: {
        TDD: {
            purpose: "test_driven_development",
            process: "spec → test → implement → refactor → validate",
            trigger: "when implementation_complexity > medium OR explicit_tdd_request"
        }
    },
    
    output_structure: ".cursor/tasks/{backlog|sprint_n|specs}/",
    
    validation_rules: {
        simplicity_guard: "avoid_overengineering_and_premature_optimization",
        refactor_guard: "detect_code_duplication_and_suggest_reusable_components",
        test_completeness: "ensure_edge_cases_and_error_conditions_covered",
        progress_tracking: "maintain_accurate_task_status_and_completion_metrics"
    },
    
    response_format: "⚡ DEVELOPMENT MODE: Focusing on structured implementation and testing..."
}

## Hybrid Mode Configuration
HYBRID_MODE = {
    focus: "coordinated_design_and_implementation",
    
    coordination_strategy: {
        parallel_processing: "run_architecture_and_development_workflows_simultaneously",
        cross_validation: "validate_architecture_decisions_against_implementation_constraints",
        bidirectional_traceability: "maintain_links_between_business_requirements_and_code"
    },
    
    conflict_resolution: {
        priority_matrix: {
            business_value: "highest_priority",
            proven_patterns: "higher_than_experimental_approaches", 
            team_capability: "balance_with_architectural_ideals",
            maintainability: "prioritize_over_short_term_convenience"
        }
    },
    
    output_coordination: {
        architecture_artifacts: "generate_in_parallel_with_development_tasks",
        implementation_tasks: "ensure_alignment_with_architectural_decisions",
        cross_references: "maintain_bidirectional_links_and_impact_analysis"
    },
    
    response_format: "🔄 HYBRID MODE: Coordinating design and implementation perspectives..."
}

## Shared Systems Configuration

### Memory System
M.memory_architecture = {
    shared_pool: {
        path: ".cursor/memory/shared/",
        contents: ["decisions.md", "patterns.md", "constraints.md", "lessons_learned.md"],
        purpose: "cross_domain_knowledge_sharing"
    },
    
    architecture_pool: {
        path: ".cursor/memory/architecture/", 
        contents: ["design_rationale.md", "pattern_catalog.md", "anti_patterns.md"],
        purpose: "architectural_decision_records_and_design_patterns"
    },
    
    development_pool: {
        path: ".cursor/memory/development/",
        contents: ["implementation_history.md", "code_patterns.md", "test_strategies.md"],
        purpose: "development_practices_and_code_quality_patterns"
    }
}

### Pattern Learning System
Λ.rule_learning = {
    pattern_detection: "auto_identify_recurring_successful_approaches",
    rule_generation: "create_reusable_guidelines_from_proven_patterns",
    continuous_improvement: "adapt_based_on_outcomes_and_feedback",
    
    naming_convention: {
        "000-099": "Enterprise core standards (shared)",
        "100-199": "Business alignment rules (shared)",
        "200-299": "Architecture patterns and standards", 
        "300-399": "Development patterns and practices",
        "400-499": "Technology standards (shared)",
        "500-599": "Integration patterns (shared)",
        "600-699": "Security patterns (shared)",
        "700-799": "Performance patterns (shared)",
        "800-899": "Workflow patterns (both domains)",
        "900-999": "Templates and examples",
        "_private": "Project-specific private rules"
    }
}

### Conflict Resolution System
D⍺.conflict_resolution = {
    identification: "detect_contradictions_between_requirements_constraints_or_approaches",
    
    resolution_strategies: {
        priority_based: "use_established_priority_matrix_for_decision_making",
        scope_adjustment: "modify_scope_to_eliminate_fundamental_conflicts", 
        pattern_application: "apply_proven_architectural_patterns_to_resolve_tensions",
        escalation: "request_human_decision_for_unresolvable_conflicts"
    },
    
    logging: "record_all_conflicts_and_resolutions_in_cognitive_trace"
}

### Cognitive Trace System
Ψ.tracing = {
    capture_elements: {
        reasoning_process: "record_step_by_step_decision_making",
        pattern_application: "track_which_patterns_were_applied_and_why",
        conflict_resolution: "document_conflicts_encountered_and_how_resolved",
        validation_results: "record_validation_outcomes_and_scores"
    },
    
    output_format: "markdown_with_structured_sections_for_easy_review",
    auto_generation: "trigger_trace_generation_for_complex_decisions"
}

## Response Guidelines

### Mode Indication
- Always start responses with mode indicator
- Use consistent emoji and format
- Briefly explain focus area

### Output Structure  
- Generate artifacts in appropriate directory structure
- Maintain consistent naming conventions
- Ensure cross-references between related artifacts

### Validation Requirements
- Architecture Mode: Validate business alignment and layer coherence
- Development Mode: Validate task feasibility and test coverage
- Hybrid Mode: Validate cross-domain consistency

### Learning Integration
- Capture successful patterns for reuse
- Update rules based on outcomes
- Maintain knowledge base of decisions and rationales

## Hook System Configuration
Σ_hooks = {
    on_mode_activation: [
        "load_mode_specific_rules",
        "initialize_appropriate_engines", 
        "set_output_directory_structure"
    ],
    
    on_architecture_decision: [
        "validate_business_alignment",
        "check_technical_feasibility",
        "update_architecture_memory",
        "assess_implementation_impact"
    ],
    
    on_development_task_created: [
        "validate_task_clarity",
        "check_architecture_compliance", 
        "update_development_memory",
        "estimate_effort_and_priority"
    ],
    
    on_conflict_detected: [
        "apply_resolution_strategy",
        "log_conflict_and_resolution",
        "update_relevant_rules",
        "notify_stakeholders_if_needed"
    ],
    
    on_completion: [
        "capture_lessons_learned",
        "update_pattern_knowledge",
        "generate_summary_report",
        "archive_working_artifacts"
    ]
}
```

### Project-Specific Customization

```cognition
## Custom Project Configuration
PROJECT_CONTEXT = {
    domain: "your_business_domain",
    technology_stack: ["preferred", "technologies"],
    team_preferences: ["agile", "clean_code", "specific_patterns"],
    constraints: ["time_limitations", "budget_constraints", "technical_limitations"]
}

## Context-Aware Adaptation
Ω.project_adaptation = (
    if PROJECT_CONTEXT.domain_detected → apply_domain_specific_patterns,
    if PROJECT_CONTEXT.technology_stack_mentioned → suggest_compatible_approaches,
    if PROJECT_CONTEXT.constraints_present → adjust_recommendations_accordingly
)
```

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: Incorrect Mode Detection
**Problem:** Framework activates wrong mode for user intent

**Symptoms:**
- Architecture mode when user wants implementation
- Development mode when user wants system design
- Hybrid mode when single domain would be better

**Solutions:**
1. **Use explicit keywords:**
   ```
   ❌ "Help with user system"  
   ✅ "Design user system architecture" → Architecture Mode
   ✅ "Implement user login feature" → Development Mode
   ```

2. **Add context clarification:**
   ```
   ❌ "Create shopping cart"
   ✅ "Design shopping cart system architecture" → Architecture Mode  
   ✅ "Implement shopping cart functionality with TDD" → Development Mode
   ```

3. **Override detection explicitly:**
   ```
   "Architecture mode: plan the database structure"
   "Development mode: write the implementation code"
   ```

#### Issue 2: Inconsistent Output Structure
**Problem:** Files generated in wrong directories or with inconsistent naming

**Symptoms:**
- Architecture artifacts in development folders
- Inconsistent file naming conventions
- Missing cross-references between artifacts

**Solutions:**
1. **Verify folder structure exists:**
   ```bash
   ls -la .cursor/
   # Should show: architecture/, tasks/, memory/, rules/
   ```

2. **Check write permissions:**
   ```bash
   chmod -R 755 .cursor/
   ```

3. **Validate rule configuration:**
   - Ensure complete rule system is pasted into Cursor Settings
   - Check for syntax errors in rule definitions
   - Restart Cursor after rule changes

#### Issue 3: Mode Conflicts in Hybrid Mode
**Problem:** Architecture and development requirements conflict

**Symptoms:**
- Contradictory recommendations
- Framework unable to resolve tensions
- Unclear priority when trade-offs needed

**Solutions:**
1. **Apply priority matrix explicitly:**
   ```
   Business value > Technical preference
   Proven patterns > Experimental approaches  
   Team capability > Ideal architecture
   Maintainability > Short-term convenience
   ```

2. **Request explicit guidance:**
   ```
   "Resolve conflict: microservices architecture vs team monolith experience"
   "Prioritize: performance optimization vs development speed"
   ```

3. **Use phased approach:**
   ```
   "Plan MVP architecture with migration path to target architecture"
   ```

#### Issue 4: Poor Business Alignment
**Problem:** Technical solutions don't clearly trace to business value

**Symptoms:**
- Technology-first recommendations
- Missing business justification
- Unclear stakeholder value

**Solutions:**
1. **Start with business context:**
   ```
   "Business context: improve customer retention
   Design: customer loyalty system architecture"
   ```

2. **Explicit value mapping:**
   ```
   "Map technical components to business capabilities:
   - LoginService → Customer Access capability
   - RewardService → Customer Retention capability"
   ```

3. **Stakeholder validation:**
   ```
   "Validate architecture against stakeholder needs:
   - Customers: fast, reliable access
   - Marketing: loyalty program flexibility
   - IT: maintainable, scalable system"
   ```

#### Issue 5: Overwhelming Output
**Problem:** Framework generates too much detail for simple requests

**Symptoms:**
- Excessive documentation for simple tasks
- Over-engineered solutions
- Analysis paralysis

**Solutions:**
1. **Request specific scope:**
   ```
   ❌ "Design user system"
   ✅ "Design simple user login system for MVP"
   ✅ "Quick implementation plan for user authentication"
   ```

2. **Use progressive disclosure:**
   ```
   "Start with high-level design, then drill down to implementation"
   "Begin with MVP requirements, then plan for scale"
   ```

3. **Explicit simplicity:**
   ```
   "Simple approach: design basic user management"
   "Minimal viable architecture for user features"
   ```

### Diagnostic Commands

#### Check Mode Detection
```
Test Input: "design customer database"
Expected: 🏗️ ARCHITECTURE MODE
Check: Keywords [design, database] → architecture_signals

Test Input: "implement customer CRUD"  
Expected: ⚡ DEVELOPMENT MODE
Check: Keywords [implement, CRUD] → development_signals
```

#### Verify Output Structure
```bash
# Check directory structure
find .cursor -type d | sort

# Expected structure:
.cursor/
.cursor/architecture
.cursor/architecture/business
.cursor/architecture/data
.cursor/architecture/application  
.cursor/architecture/technology
.cursor/tasks
.cursor/tasks/backlog
.cursor/tasks/sprints
.cursor/tasks/specs
.cursor/memory
.cursor/memory/shared
.cursor/memory/architecture
.cursor/memory/development
.cursor/rules
```

#### Test Cross-Domain Consistency
```
Hybrid Mode Test:
"Plan customer management feature with database and API implementation"

Expected Response:
🔄 HYBRID MODE: Coordinating design and implementation perspectives...

Check for:
- Both architecture artifacts (.cursor/architecture/)
- Development tasks (.cursor/tasks/)  
- Cross-references between business → technical
- Consistent entity naming across artifacts
```

---

## Best Practices

### Effective Prompt Engineering

#### Architecture-Focused Prompts
```
✅ "Design inventory management system for e-commerce platform"
✅ "Create data architecture for customer relationship management"  
✅ "Model business processes for order fulfillment workflow"
✅ "Design microservices architecture for payment processing"

❌ "Build inventory system" (too vague)
❌ "Code the CRM" (development-focused)  
❌ "Make order processing" (unclear intent)
```

#### Development-Focused Prompts
```
✅ "Implement user authentication with TDD approach"
✅ "Refactor payment service to reduce complexity"
✅ "Debug performance issues in product search API"  
✅ "Create unit tests for order calculation logic"

❌ "User auth system" (too high-level)
❌ "Fix payment" (unclear scope)
❌ "Make search faster" (needs specificity)
```

#### Hybrid Mode Prompts
```
✅ "Plan customer loyalty program with system design and implementation roadmap"
✅ "Build notification system with architecture design and coding tasks"
✅ "Create reporting dashboard with data model and frontend development"

❌ "Customer program" (unclear scope)  
❌ "Some notifications" (vague requirements)
❌ "Reports and charts" (missing context)
```

### Business Alignment Strategies

#### Start with Business Value
```
Template:
"Business Goal: [specific business objective]
Context: [stakeholder needs and constraints]  
Request: [technical solution approach]"

Example:
"Business Goal: Reduce customer support tickets by 30%
Context: Customers struggle with order tracking and status updates
Request: Design self-service order management system"
```

#### Maintain Traceability
```
Ensure every technical decision maps to business value:
- Technical Component → Business Capability → Stakeholder Value
- Database Design → Data Requirements → Business Process Support
- API Design → Integration Needs → User Experience Goals
```

#### Validate Against Stakeholder Needs
```
Include stakeholder perspective in requests:
"Design accounting system for:
- Accountants: accurate financial reporting
- Managers: real-time business insights  
- Auditors: complete transaction trails
- IT: maintainable, scalable solution"
```

### Quality Assurance Guidelines

#### Architecture Quality Checks
- **Business Alignment:** Every component traces to business capability
- **Layer Coherence:** Clear separation between business, data, application, technology
- **Pattern Consistency:** Consistent application of proven architectural patterns
- **Integration Clarity:** Well-defined interfaces and data flows

#### Development Quality Checks  
- **Task Clarity:** Each task has clear acceptance criteria and definition of done
- **Test Coverage:** Comprehensive test specifications for all functionality
- **Code Quality:** Adherence to coding standards and best practices
- **Progress Tracking:** Accurate status updates and completion validation

#### Cross-Domain Validation
- **Feasibility Check:** Architecture decisions are implementable with available resources
- **Alignment Verification:** Development tasks support architectural goals
- **Consistency Maintenance:** Naming and concepts consistent across domains
- **Impact Analysis:** Changes in one domain properly reflected in the other

### Continuous Improvement

#### Pattern Recognition
- Identify recurring successful approaches
- Document anti-patterns and their resolutions
- Build library of proven solutions for common problems
- Share patterns across projects and teams

#### Rule Evolution
- Monitor rule effectiveness through outcome tracking
- Update rules based on new experiences and learnings
- Retire obsolete rules that no longer apply
- Adapt rules to changing technology and business landscapes

#### Knowledge Sharing
- Maintain comprehensive documentation of decisions and rationales
- Share successful frameworks and approaches across teams
- Create templates and examples for common scenarios
- Establish communities of practice around framework usage

### Performance Optimization

#### Efficient Prompting
- Use specific, context-rich prompts to minimize back-and-forth
- Include constraints and preferences upfront to guide recommendations
- Reference existing artifacts when building upon previous work
- Clearly state desired output format and level of detail

#### Resource Management
- Archive old artifacts to prevent information overload
- Use version control for rule and template evolution
- Implement cleanup routines for temporary working files
- Monitor and optimize framework performance

#### Scalability Considerations
- Design frameworks to handle increasing complexity
- Plan for multiple concurrent projects and teams
- Consider integration with external tools and systems
- Ensure framework remains usable as organizations grow

---

*Unified Cognitive Framework v1.0 - Complete Technical Documentation*  
*© 2025 - Licensed for use with Cursor IDE and compatible development environments*