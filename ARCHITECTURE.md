# ARCHITECTURE.md

## 1. Executive Summary

Planolo is a consumer SaaS platform that transforms financial planning from static spreadsheets into interactive life simulations, combining deterministic CPF/tax/housing calculations with Monte Carlo market modeling and AI-powered plain-language explanations. The product launches in Singapore—leveraging the market's digital sophistication, CPF complexity, and property-centric culture—before expanding across Southeast Asia's emerging middle class.

## 2. Component Diagram

```mermaid
graph TD
    subgraph "Client Layer"
        A[React 18 + TypeScript<br/>Progressive Web App]
        B[Scenario Canvas<br/>D3.js Timeline]
        C[Chat Interface<br/>WebSocket]
        D[shadcn/ui Components<br/>Radix Primitives]
        E[Zustand Store<br/>Scenario State]
        F[TanStack Query<br/>API Cache]
    end

    subgraph "Edge Layer"
        G1[CloudFront CDN<br/>Static Assets]
        G2[WAF<br/>DDoS Protection]
        G3[API Gateway<br/>Rate Limiting]
    end

    subgraph "API Gateway"
        G[FastAPI<br/>Async Python 3.11+]
        H[WebSocket Server<br/>Chat & Live Updates]
        I[REST Endpoints<br/>CRUD + Simulation]
        J[Auth Middleware<br/>JWT + Refresh]
    end

    subgraph "Simulation Core"
        K[Deterministic Engine<br/>Pandas + NumPy]
        L[CPF Calculator<br/>OA/SA/MA Rules]
        M[Tax Engine<br/>IRAS Logic]
        N[Housing Calculator<br/>HDB/Bank Loans]
        O[Monte Carlo Engine<br/>Ray Distributed]
        P[Market Simulator<br/>GBM + Jump Diffusion]
    end

    subgraph "AI/ML Layer"
        Q[Llama-3-8B<br/>Local via Ollama]
        R[Mixtral 8x22B<br/>vLLM on A10G]
        S[Premium Models<br/>Claude 3.5 Sonnet]
        T[RAG Pipeline<br/>LangChain + Qdrant]
        U[Prompt Cache<br/>Semantic Dedup]
    end

    subgraph "Data Persistence"
        V[PostgreSQL 15<br/>User Data + Scenarios]
        W[Redis 7<br/>Session + Results]
        X[Snowflake<br/>Market Data ETL]
        Y[Qdrant<br/>600M CPF/Tax Vectors]
        Z[S3<br/>Export Archive]
    end

    subgraph "Infrastructure"
        AA[AWS EKS<br/>Kubernetes 1.28]
        AB[ALB + CloudFront<br/>CDN + SSL]
        AC[Secrets Manager<br/>API Keys + Certs]
        AD[CloudWatch<br/>Logs + Metrics]
        AE[X-Ray<br/>Distributed Tracing]
    end

    A -->|HTTPS| G1
    G1 --> G2
    G2 --> G3
    G3 --> AB
    AB --> G
    B --> I
    C -->|WSS| H
    E <--> F
    F <--> I
    
    G --> J
    J --> K
    J --> O
    
    K --> L
    K --> M
    K --> N
    
    O --> P
    P -->|10k paths| O
    
    I -->|Results| W
    K -->|Deterministic| W
    O -->|Probabilistic| W
    
    H --> T
    T --> Q
    T -->|Pro users| R
    T -->|Plus users| S
    
    T <--> Y
    Q --> U
    R --> U
    S --> U
    
    G --> V
    P --> X
    I -->|Exports| Z
    
    G -->|Runs on| AA
    AA --> AC
    AA --> AD
    AA --> AE
```

## 3. Runtime Data-Flow

```mermaid
sequenceDiagram
    participant U as User
    participant CDN as CloudFront
    participant WAF as WAF
    participant F as Frontend
    participant WS as WebSocket
    participant API as FastAPI
    participant Auth as Auth Layer
    participant Cache as Redis
    participant Queue as SQS
    participant DE as Deterministic Engine
    participant MC as Monte Carlo Engine
    participant AI as AI Layer
    participant RAG as RAG Pipeline
    participant DB as PostgreSQL
    participant Trace as X-Ray

    Note over U,Trace: Scenario Update Flow with Full Observability
    U->>CDN: HTTPS Request
    CDN->>WAF: Security Check
    WAF->>F: Serve React App
    F->>F: Initialize PWA
    
    U->>F: Drags income change at age 35
    F->>F: Optimistic UI update
    F->>API: PUT /api/scenarios/{id}/events
    API->>Trace: Start trace span
    API->>Auth: Verify JWT
    Auth->>API: User context + permissions
    API->>Cache: Check result cache (hash key)
    
    alt Cache Miss
        API->>DE: Calculate new cashflows
        DE->>Trace: Log calculation time
        DE->>DE: Apply CPF rules (2.5% OA, 4% SA)
        DE->>DE: Apply tax rules (progressive brackets)
        DE->>DE: Apply housing rules (LTV limits)
        DE->>API: Deterministic results
        
        opt User has Pro subscription
            API->>Queue: Enqueue MC job
            Queue->>MC: Process with priority
            MC->>MC: Load market assumptions
            MC->>MC: Sample 10k inflation paths
            MC->>MC: Sample 10k return paths
            MC->>MC: Calculate percentiles
            MC->>Trace: Log simulation metrics
            MC->>API: p10/p50/p90 bands
        end
        
        API->>Cache: Store results (TTL 24h)
        API->>DB: Persist scenario (ACID)
        DB->>DB: Trigger audit log
    end
    
    API->>Trace: End trace span
    API->>F: Return results + ETag
    F->>U: Update charts (60fps)

    Note over U,Trace: AI Explanation Flow with Safety Checks
    U->>F: Clicks "Explain it"
    F->>API: POST /api/explain
    API->>API: Sanitize input
    API->>Cache: Check explanation cache
    
    alt Explanation not cached
        API->>RAG: Query relevant docs
        RAG->>RAG: Embed user context
        RAG->>RAG: Vector similarity search
        RAG->>AI: Generate with guardrails
        
        alt Free tier
            AI->>AI: Route to local Llama-3
            AI->>AI: 4-bit quantization
        else Pro tier
            AI->>AI: Route to Mixtral GPU
            AI->>AI: Batched inference
        else Plus tier
            AI->>AI: Route to Claude 3.5
            AI->>AI: Enhanced context
        end
        
        AI->>AI: Validate no hallucinations
        AI->>API: Narrative + confidence
        API->>Cache: Store explanation
    end
    
    API->>F: Return narrative
    F->>U: Display in modal

    Note over U,Trace: Chat Interaction with State Management
    U->>F: "What if I retire at 55?"
    F->>WS: Send message
    WS->>Auth: Verify connection token
    WS->>WS: Rate limit check
    WS->>RAG: Retrieve scenario context
    RAG->>AI: Generate response plan
    AI->>DE: Trigger new calculation
    DE->>AI: Updated numbers
    AI->>WS: Stream response tokens
    WS->>F: Real-time display
    F->>U: Show typing indicator
    WS->>DB: Log conversation
    DB->>Trace: Async audit trail
```

## 4. Tech Stack Rationale

| Layer | Framework | Why it beats alternatives | Performance Impact |
|-------|-----------|---------------------------|-------------------|
| **Frontend** | React 18 + TypeScript | Concurrent rendering for 60fps timeline drag, vast ecosystem, strict typing prevents financial calculation bugs | -40% TTI vs Angular |
| **UI Library** | shadcn/ui on Radix | No vendor lock, accessible by default, tree-shakeable vs Material-UI bloat | 70KB vs 300KB bundle |
| **State** | Zustand | 8KB vs Redux 60KB, no boilerplate, TypeScript inference just works | 5ms vs 15ms updates |
| **Data Fetching** | TanStack Query | Built-in cache invalidation, optimistic updates, offline support beats SWR | 90% cache hit rate |
| **CDN** | CloudFront | 15ms Singapore latency, $0.02/GB vs $0.08 competitors, WebSocket support | 300ms → 50ms global |
| **WAF** | AWS WAF | Managed rules, geo-blocking, rate limiting at edge | Blocks 99.9% attacks |
| **API** | FastAPI | Native async beats Flask, automatic OpenAPI, Pydantic validation > Express | 10k req/s capacity |
| **Calculations** | Pandas + NumPy | Vectorized ops 100x faster than loops, battle-tested in quant finance | 50ms for 50yr projection |
| **Distributed Compute** | Ray | Scales laptop→cluster seamlessly, 10x faster than multiprocessing for MC | 5s for 10k paths |
| **Job Queue** | SQS FIFO | Exactly-once processing, dead letter queues, $0.40/million messages | 99.999% reliability |
| **Local AI** | Llama-3-8B via Ollama | 4-bit quantization fits 6GB VRAM, quality matches GPT-3.5 | 300ms first token |
| **GPU Inference** | vLLM | 24x throughput vs HuggingFace, PagedAttention for long context | 2000 tokens/sec |
| **Cloud AI** | Mixtral 8x22B | MoE routes to 2 experts, 4x cheaper than GPT-4 at similar quality | $0.002/1k tokens |
| **Vector Search** | Qdrant | Rust performance, on-premise option, filtering beats Pinecone | 10ms @ 1M vectors |
| **Database** | PostgreSQL 15 | JSONB for flexible schemas, row-level security, ACID critical for money | 1ms queries |
| **Cache** | Redis 7 | Redis Functions for atomic updates, Streams for event sourcing | 0.1ms latency |
| **Analytics DB** | Snowflake | Zero-copy cloning for backtesting, native time-series, pay-per-query | $2/TB scanned |
| **Container Platform** | EKS | Spot instances for GPU nodes save 70%, Karpenter autoscaling | 30s scale-up |
| **Monitoring** | X-Ray + CloudWatch | Distributed tracing, custom metrics, anomaly detection | 2% overhead |
| **IaC** | Terraform | Type-safe with CDK, state locking, provider ecosystem | 5min provisions |

## 5. Repository Layout Proposal

```
planolo/
├── backend/
│   ├── api/
│   │   ├── endpoints/        # FastAPI route handlers
│   │   ├── middleware/       # Auth, rate limiting, logging
│   │   ├── websocket/        # Chat and live update handlers
│   │   └── dependencies/     # Shared request dependencies
│   ├── core/
│   │   ├── config.py         # Settings via Pydantic BaseSettings
│   │   ├── security.py       # JWT, password hashing, encryption
│   │   ├── database.py       # Connection pooling, transactions
│   │   └── exceptions.py     # Custom exception hierarchy
│   ├── models/
│   │   ├── domain/           # Scenario, User, Subscription models
│   │   ├── schemas/          # Request/response Pydantic models
│   │   └── events/           # Event sourcing definitions
│   ├── services/
│   │   ├── simulation/       # Deterministic and MC engines
│   │   ├── ai/               # LLM orchestration, RAG pipeline
│   │   ├── export/           # CSV, PDF generation
│   │   └── audit/            # Compliance logging
│   ├── workers/
│   │   ├── monte_carlo.py    # Ray distributed workers
│   │   └── export_queue.py   # Background job processors
│   └── tests/                # Pytest suites with fixtures
│
├── frontend/
│   ├── src/
│   │   ├── components/       # React components (atoms → organisms)
│   │   ├── features/         # Feature-based modules
│   │   ├── hooks/            # Custom React hooks
│   │   ├── stores/           # Zustand store slices
│   │   ├── api/              # Generated TypeScript client
│   │   ├── utils/            # Helpers, formatters, constants
│   │   └── workers/          # Web Workers for heavy compute
│   ├── public/               # Static assets, PWA manifest
│   └── e2e/                  # Playwright test specs
│
├── sim_core/                 # Standalone simulation package
│   ├── calculators/
│   │   ├── cpf.py            # CPF contribution/interest logic
│   │   ├── tax.py            # IRAS tax calculation
│   │   ├── housing.py        # HDB/bank loan amortization
│   │   └── portfolio.py      # Investment returns modeling
│   ├── monte_carlo/
│   │   ├── market_model.py   # GBM, jump diffusion, regime switching
│   │   ├── ray_runner.py     # Distributed simulation orchestrator
│   │   └── aggregator.py     # Percentile band calculator
│   ├── validators/           # Input validation and bounds checking
│   └── tests/                # Property-based tests with Hypothesis
│
├── infra/
│   ├── terraform/
│   │   ├── modules/          # Reusable infrastructure components
│   │   ├── environments/     # Dev, staging, prod configurations
│   │   └── backend.tf        # S3 state backend config
│   ├── k8s/
│   │   ├── base/             # Namespace, RBAC, network policies
│   │   ├── apps/             # Deployment manifests
│   │   ├── monitoring/       # Prometheus, Grafana, Loki
│   │   └── security/         # Pod security policies, OPA
│   ├── docker/
│   │   ├── backend.Dockerfile
│   │   ├── frontend.Dockerfile
│   │   └── llama.Dockerfile
│   └── helm/
│       └── planolo/          # Helm chart for full stack
│
├── scripts/
│   ├── setup_dev.sh          # Install deps, seed data
│   ├── run_migrations.py     # Alembic migration runner
│   ├── seed_market_data.py   # Populate Snowflake tables
│   ├── load_test.py          # Locust performance tests
│   └── security_scan.sh      # OWASP ZAP automation
│
├── docs/
│   ├── architecture/         # ADRs, design docs
│   ├── api/                  # OpenAPI spec, examples
│   ├── deployment/           # Runbooks, disaster recovery
│   └── compliance/           # PDPA, security policies
│
├── .github/
│   ├── workflows/
│   │   ├── ci.yml            # Test, lint, type-check
│   │   ├── cd.yml            # Build, push, deploy
│   │   ├── security.yml      # Dependency scanning, SAST
│   │   └── load-test.yml     # Scheduled performance tests
│   ├── CODEOWNERS
│   └── dependabot.yml
│
├── monitoring/
│   ├── dashboards/           # Grafana JSON exports
│   ├── alerts/               # Prometheus alert rules
│   └── slos/                 # Service level objectives
│
├── .devcontainer/
│   ├── devcontainer.json     # VS Code configuration
│   ├── Dockerfile            # Dev environment image
│   └── features/             # Dev container features
│
├── docker-compose.yml        # Local stack definition
├── docker-compose.test.yml   # Test environment
├── Makefile                  # Common development tasks
├── pyproject.toml            # Python project config (Poetry)
├── package.json              # Frontend dependencies (pnpm)
├── .env.example              # Environment variable template
├── .pre-commit-config.yaml   # Git hooks configuration
└── README.md                 # Getting started guide
```

## 6. Non-Functional SLAs

### Performance Targets

| Metric | Target | Measurement | Alert Threshold |
|--------|--------|-------------|-----------------|
| API p50 latency | <100ms | Deterministic calculations | >150ms |
| API p95 latency | <200ms | Including cache lookup | >300ms |
| API p99 latency | <500ms | Cold start scenarios | >750ms |
| Monte Carlo completion | <5s | 10,000 paths with Ray | >7s |
| Frontend TTI | <2s | Time to Interactive on 4G | >3s |
| Timeline drag FPS | 60fps | Chrome DevTools measurement | <30fps |
| WebSocket latency | <50ms | Singapore region | >100ms |
| AI response time (local) | <3s | Llama-3 first token | >5s |
| AI response time (GPU) | <1s | Mixtral first token | >2s |
| Database query p95 | <10ms | Complex queries with indexes | >20ms |
| Cache hit ratio | >90% | Redis hit rate | <80% |
| Concurrent users | 10,000 | Per cluster with autoscaling | N/A |
| Error rate | <0.1% | 5xx responses | >0.5% |
| Availability | 99.9% | Uptime per month | <99.5% |

### Security Requirements

| Category | Implementation | Details | Validation |
|----------|----------------|---------|------------|
| Data at rest | AES-256-GCM | PostgreSQL TDE + S3 SSE-KMS | Quarterly audit |
| Data in transit | TLS 1.3 | HSTS, cert pinning, no downgrades | SSL Labs A+ |
| Authentication | JWT + Refresh | 15min access, 7d refresh, rotation | OWASP ASVS L2 |
| Authorization | RBAC + ABAC | PostgreSQL RLS + OPA policies | Penetration test |
| Secrets | AWS Secrets Manager | Auto rotation 90d, versioning | Access logs |
| API Security | Rate limiting | 100/min free, 1000/min pro, DDoS protection | Load test |
| Input validation | Pydantic + OWASP | Parameterized queries, XSS prevention | SAST scan |
| Audit logging | CloudWatch + S3 | Immutable logs, 1yr retention, SIEM ready | Compliance review |
| PII handling | Field encryption | Separate keys per user, GDPR compliant | Data flow audit |
| Session management | Redis + secure cookies | HttpOnly, SameSite, CSRF tokens | Security headers |
| Dependency scanning | Snyk + Dependabot | Daily scans, auto PRs for patches | <24h patch SLA |
| Infrastructure | IMDSv2, VPC endpoints | No public subnets for data, NACLs | AWS Well-Architected |

### Availability & Reliability

| Aspect | Target | Implementation | Testing |
|--------|--------|----------------|---------|
| Uptime SLA | 99.9% | Multi-AZ, auto-failover, health checks | Chaos engineering |
| RTO | <1 hour | Automated recovery runbooks | Quarterly drills |
| RPO | <5 minutes | Continuous replication, point-in-time recovery | Backup restore test |
| Degraded mode | Core features only | Circuit breakers, fallback to cache | Feature flags |
| Data durability | 99.999999999% | S3 cross-region replication | Corruption checks |
| Disaster recovery | <4 hours | Standby region (Mumbai), Terraform | Annual DR test |
| Rollback capability | <15 minutes | Blue-green deployment, database snapshots | Every deployment |
| Monitoring coverage | 100% critical paths | Distributed tracing, custom metrics | Alert accuracy |
| Incident response | <5 min acknowledge | PagerDuty escalation, runbooks | On-call drills |

### Scalability Limits

| Resource | Limit | Scaling Strategy | Cost Impact |
|----------|-------|------------------|-------------|
| API pods | 2-100 | HPA on CPU/memory, 30s scale | $0.10/pod/hour |
| GPU nodes | 0-10 | Karpenter with 5min provision | $1.20/hour A10G |
| Database connections | 1000 | PgBouncer pooling, read replicas | $500/month RDS |
| Redis memory | 128GB | Cluster mode, eviction policies | $1000/month |
| S3 storage | Unlimited | Lifecycle policies, Glacier after 90d | $0.023/GB/month |
| CDN bandwidth | 100TB/month | Multiple origins, compression | $0.02/GB |
| WebSocket connections | 50k/server | Sticky sessions, connection draining | $200/server |
| Background jobs | 10k/minute | SQS with Lambda consumers | $0.40/million |

## 7. Initial Backlog (MoSCoW)

### Must Have - Month 0-1: Core Calculation Engine

**M01: Singapore CPF Calculator with Full Compliance**
```python
# Acceptance criteria pseudocode
def test_cpf_calculation():
    assert cpf_oa_rate(age=35, balance=50000) == 0.025  # 2.5%
    assert cpf_sa_rate(age=35, balance=50000) == 0.04   # 4.0%
    assert cpf_ma_rate(age=35, balance=50000) == 0.04   # 4.0%
    assert employer_contribution(salary=5000, age=35) == 850  # 17%
    assert employee_contribution(salary=5000, age=35) == 1000 # 20%
    assert contribution_ceiling() == 6800  # Monthly cap
    # Must match cpf.gov.sg calculator within $1 for 100 test cases
```

**M02: HDB Resale Calculator with Grants**
```python
# Key features to implement:
- API integration with data.gov.sg resale prices
- Loan eligibility: 75% LTV or 55% for second property
- Downpayment: 25% (20% CPF + 5% cash minimum)  
- Stamp duty: 1% first $180k, 2% next $180k, 3% remainder
- Grants: Enhanced Housing Grant, Proximity Grant calculations
- Validation: Must handle all HDB towns and flat types
```

**M03: Singapore Tax Engine YA 2024**
```python
# Progressive brackets and reliefs:
- Tax brackets: 0-2%, 3.5%, 7%, 11.5%, 15%, 18%, 19%, 20%, 22%
- Earned income relief: $1,000 (<55yo), $6,000 (55-59yo), $8,000 (60+yo)
- CPF relief: Capped at $37,740 (employee + employer)
- NS relief: $3,000/$1,500 for NSmen/parents
- Must generate Form B equivalent output
```

**M04: Scenario Data Model with Event Sourcing**
```sql
-- Core tables
CREATE TABLE scenarios (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    name TEXT NOT NULL,
    events JSONB NOT NULL, -- Array of timestamped events
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    version INTEGER DEFAULT 1,
    is_deleted BOOLEAN DEFAULT FALSE
);

CREATE TABLE scenario_results (
    id UUID PRIMARY KEY,
    scenario_id UUID REFERENCES scenarios,
    calculation_type TEXT NOT NULL, -- 'deterministic' or 'monte_carlo'
    results JSONB NOT NULL, -- Compressed results
    cache_key TEXT UNIQUE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    expires_at TIMESTAMPTZ
);

-- Event sourcing for audit
CREATE TABLE scenario_events (
    id UUID PRIMARY KEY,
    scenario_id UUID,
    event_type TEXT NOT NULL,
    event_data JSONB NOT NULL,
    user_id UUID NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Should Have - Month 1-2: Minimal UI

**S01: Interactive Timeline Component**
```typescript
// React component with D3.js
interface TimelineEvent {
  id: string;
  type: 'income' | 'expense' | 'life_event';
  age: number;
  amount?: number;
  description: string;
}

// Requirements:
- Drag events with 60fps performance
- Snap to year boundaries
- Collision detection for overlapping events
- Touch support for mobile
- Zoom/pan with mousewheel
- Undo/redo support
```

**S02: Real-time Chart Visualization**
```typescript
// Using Recharts with React.memo optimization
- Stacked area: income, expenses, savings
- Line chart: net worth projection
- Confidence bands for Monte Carlo results
- Responsive design with aspect ratio preservation
- Export as PNG/SVG
- Maximum 100ms render time for 50-year projection
```

**S03: CSV Export with Detailed Breakdown**
```python
# Export format:
Year,Age,Income,CPF_OA,CPF_SA,CPF_MA,Tax,Housing,Expenses,Savings,Net_Worth
2024,35,120000,24000,18000,6000,15000,12000,48000,21000,250000
# Include all calculation details
# UTF-8 BOM for Excel compatibility
# Streaming generation for large datasets
```

**S04: RESTful API with OpenAPI Spec**
```python
# FastAPI endpoints:
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/refresh
GET    /api/scenarios
POST   /api/scenarios
PUT    /api/scenarios/{id}
DELETE /api/scenarios/{id}
POST   /api/scenarios/{id}/calculate
POST   /api/scenarios/{id}/explain
GET    /api/scenarios/{id}/export
# Auto-generated TypeScript client
# Request/response validation
# Structured error responses
```

### Could Have - Month 2: AI Integration

**C01: Llama-3 Integration with Guardrails**
```python
# Safety measures:
- Token limit: 1000 input, 500 output
- Prompt injection detection
- Output validation against calculations
- Confidence scoring
- Fallback to template responses
- Response time <3s with timeout
```

**C02: Intelligent Caching Strategy**
```python
# Redis caching layers:
- L1: Calculation results (24h TTL)
- L2: AI explanations (7d TTL)
- L3: User sessions (15min sliding)
- Cache warming for common scenarios
- Invalidation on rule changes
- Compression for large results
```

**C03: Enterprise Authentication**
```python
# Security features:
- Argon2id password hashing
- JWT with RS256 signing
- Refresh token rotation
- 2FA via TOTP
- OAuth2 social login
- Account lockout after 5 attempts
- Password complexity requirements
```

**C04: CI/CD Excellence**
```yaml
# GitHub Actions pipeline:
- Parallel test execution
- Coverage enforcement (>80%)
- Security scanning (Snyk, CodeQL)
- Performance regression tests
- Automated changelog
- Semantic versioning
- Canary deployment (10% traffic)
- Automatic rollback on errors
```

### Won't Have - Defer Post-PMF

| Feature | Reason for Deferral | Revisit Condition |
|---------|--------------------|--------------------|
| Mobile app | PWA sufficient, saves 3 months | 10k MAU |
| Voice interface | Waiting for Whisper-quality OSS | Better models |
| Multi-country | Singapore complexity first | 1k paid users |
| Real-time prices | Static modeling adequate | Enterprise demand |
| Blockchain | No clear user value | Regulatory clarity |

## 8. Open Risks & Mitigations

| Risk | Probability | Impact | Mitigation Strategy | Success Metric | Contingency |
|------|-------------|---------|-------------------|----------------|-------------|
| **MAS regulation** | Medium | Critical | Educational positioning; no "advice" language; disclaimers on every page; prepare sandbox application | Legal review passed | Pivot to B2B2C |
| **LLM hallucinations** | High | High | Deterministic validation; confidence scores; "I don't know" responses; human-in-loop for edge cases | <0.1% wrong calculations | Disable AI tier |
| **GPU cost spike** | Medium | Medium | Spot instances with fallback; request batching; cache everything; usage-based pricing | <$0.01/request | CPU inference |
| **Data breach** | Low | Critical | Zero-trust architecture; encryption everywhere; bug bounty; cyber insurance; incident response plan | Pass pen test | Kill switch ready |
| **Scope creep** | High | High | Public roadmap; "ideas" parking lot; user voting system; monthly reviews | 2-week sprints | Cut features |
| **Key person risk** | High | Critical | Document everything; pair programming; recorded architecture sessions; bus factor >1 | <1 day handover | Hire contractor |
| **Market timing** | Medium | High | Launch during bonus season; financial anxiety triggers; content marketing | 100 signups/week | Extend runway |
| **Technical debt** | High | Medium | 20% time for refactoring; automated code quality; architecture reviews | <20% debt ratio | Freeze features |
| **Competitor** | High | Low | Ship weekly; community building; unique AI features; Singapore focus | 2x feature velocity | Open source |

## 9. Getting Started

### Production-Ready Dev Container (`.devcontainer/devcontainer.json`)

```json
{
  "name": "Planolo Full-Stack Development",
  "dockerComposeFile": "../docker-compose.yml",
  "service": "devcontainer",
  "workspaceFolder": "/workspace",
  "shutdownAction": "stopCompose",
  
  "features": {
    "ghcr.io/devcontainers/features/python:1": {
      "version": "3.11",
      "installTools": true,
      "toolsToInstall": "poetry,black,ruff,mypy"
    },
    "ghcr.io/devcontainers/features/node:1": {
      "version": "20",
      "nodeGypDependencies": true,
      "installYarnGlobally": false
    },
    "ghcr.io/devcontainers/features/docker-in-docker:2": {
      "version": "latest",
      "dockerDashComposeVersion": "v2"
    },
    "ghcr.io/devcontainers/features/aws-cli:1": {
      "version": "latest"
    },
    "ghcr.io/devcontainers/features/terraform:1": {
      "version": "1.6",
      "installTFSwitch": true
    },
    "ghcr.io/devcontainers/features/kubectl-helm-minikube:1": {
      "kubectl": "latest",
      "helm": "latest",
      "minikube": "none"
    },
    "ghcr.io/devcontainers/features/rust:1": {
      "version": "stable",
      "profile": "minimal"
    }
  },
  
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "ms-python.black-formatter",
        "charliermarsh.ruff",
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "bradlc.vscode-tailwindcss",
        "Prisma.prisma",
        "HashiCorp.terraform",
        "ms-kubernetes-tools.vscode-kubernetes-tools",
        "ms-azuretools.vscode-docker",
        "eamodio.gitlens",
        "usernamehw.errorlens",
        "yzhang.markdown-all-in-one",
        "redhat.vscode-yaml",
        "humao.rest-client",
        "Gruntfuggly.todo-tree",
        "wayou.vscode-todo-highlight",
        "mikestead.dotenv",
        "oderwat.indent-rainbow"
      ],
      "settings": {
        "python.defaultInterpreterPath": "/workspace/.venv/bin/python",
        "python.terminal.activateEnvironment": true,
        "python.formatting.provider": "black",
        "python.linting.enabled": true,
        "python.linting.ruffEnabled": true,
        "python.testing.pytestEnabled": true,
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
          "source.organizeImports": true,
          "source.fixAll": true
        },
        "typescript.preferences.importModuleSpecifier": "relative",
        "typescript.updateImportsOnFileMove.enabled": "always",
        "eslint.validate": ["javascript", "typescript", "javascriptreact", "typescriptreact"],
        "tailwindCSS.experimental.classRegex": [
          ["cva\\(([^)]*)\\)", "[\"'`]([^\"'`]*).*?[\"'`]"],
          ["cx\\(([^)]*)\\)", "[\"'`]([^\"'`]*).*?[\"'`]"]
        ],
        "files.exclude": {
          "**/__pycache__": true,
          "**/.pytest_cache": true,
          "**/.mypy_cache": true,
          "**/node_modules": true,
          "**/.next": true
        },
        "todo-tree.tree.showBadges": true,
        "todo-tree.tree.showCountsInTree": true
      }
    }
  },
  
  "forwardPorts": [3000, 8000, 5432, 6379, 11434, 9000, 1080],
  "portsAttributes": {
    "3000": {"label": "Frontend", "onAutoForward": "openBrowser"},
    "8000": {"label": "Backend API", "onAutoForward": "openBrowser"},
    "5432": {"label": "PostgreSQL"},
    "6379": {"label": "Redis"},
    "11434": {"label": "Ollama"},
    "9000": {"label": "MinIO S3"},
    "1080": {"label": "MailDev"}
  },
  
  "postCreateCommand": "make dev-setup",
  "postStartCommand": "make dev-services",
  "postAttachCommand": "make dev-status",
  
  "remoteUser": "vscode",
  "mounts": [
    "source=${localEnv:HOME}/.ssh,target=/home/vscode/.ssh,type=bind,consistency=cached",
    "source=${localEnv:HOME}/.aws,target=/home/vscode/.aws,type=bind,consistency=cached",
    "source=planolo-node-modules,target=/workspace/frontend/node_modules,type=volume",
    "source=planolo-python-venv,target=/workspace/.venv,type=volume",
    "source=planolo-postgres-data,target=/var/lib/postgresql/data,type=volume",
    "source=planolo-redis-data,target=/data,type=volume"
  ],
  
  "runArgs": [
    "--init",
    "--privileged",
    "--network=host",
    "--add-host=host.docker.internal:host-gateway"
  ],
  
  "env": {
    "PYTHONUNBUFFERED": "1",
    "POETRY_VIRTUALENVS_IN_PROJECT": "true",
    "NODE_ENV": "development",
    "PYTHONDONTWRITEBYTECODE": "1",
    "TERM": "xterm-256color"
  },
  
  "waitFor": "postCreateCommand",
  "updateContentCommand": "make dev-update"
}
```

### Production-Grade Docker Compose (`docker-compose.yml`)

```yaml
version: '3.8'

x-backend-env: &backend-env
  DATABASE_URL: postgresql://planolo:localdev123@postgres:5432/planolo_dev
  REDIS_URL: redis://redis:6379/0
  JWT_SECRET: ${JWT_SECRET:-local-development-secret-change-in-production}
  JWT_ALGORITHM: RS256
  OLLAMA_HOST: http://llama:11434
  S3_ENDPOINT_URL: http://minio:9000
  S3_ACCESS_KEY: minioadmin
  S3_SECRET_KEY: minioadmin
  S3_BUCKET: planolo-exports
  SMTP_HOST: maildev
  SMTP_PORT: 1025
  ENVIRONMENT: development
  LOG_LEVEL: ${LOG_LEVEL:-DEBUG}
  PYTHONUNBUFFERED: 1
  OTEL_EXPORTER_OTLP_ENDPOINT: http://jaeger:4318

x-healthcheck-defaults: &healthcheck-defaults
  interval: 10s
  timeout: 5s
  retries: 5
  start_period: 30s

services:
  devcontainer:
    build:
      context: .
      dockerfile: .devcontainer/Dockerfile
      args:
        PYTHON_VERSION: 3.11
        NODE_VERSION: 20
    volumes:
      - ..:/workspace:cached
      - ~/.ssh:/home/vscode/.ssh:ro
      - ~/.aws:/home/vscode/.aws:ro
      - /var/run/docker.sock:/var/run/docker.sock
      - vscode-extensions:/home/vscode/.vscode-server/extensions
    environment:
      DISPLAY: ${DISPLAY:-host.docker.internal:0}
    command: sleep infinity
    network_mode: host

  postgres:
    image: postgres:15-alpine
    restart: unless-stopped
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: planolo
      POSTGRES_PASSWORD: localdev123
      POSTGRES_DB: planolo_dev
      POSTGRES_INITDB_ARGS: >-
        -c shared_preload_libraries=pg_stat_statements,pgaudit
        -c log_statement=all
        -c log_connections=on
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/001-init.sql:ro
      - ./scripts/seed-data.sql:/docker-entrypoint-initdb.d/002-seed.sql:ro
    healthcheck:
      <<: *healthcheck-defaults
      test: ["CMD-SHELL", "pg_isready -U planolo -d planolo_dev"]
    deploy:
      resources:
        limits:
          memory: 1G
        reservations:
          memory: 512M

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    ports:
      - "6379:6379"
    command: >
      redis-server
      --appendonly yes
      --appendfilename "redis.aof"
      --appendfsync everysec
      --save 60 1000
      --save 300 10
      --save 900 1
      --maxmemory 512mb
      --maxmemory-policy allkeys-lru
      --protected-mode no
      --tcp-backlog 511
      --timeout 0
      --tcp-keepalive 300
    volumes:
      - redis_data:/data
    sysctls:
      - net.core.somaxconn=511
    healthcheck:
      <<: *healthcheck-defaults
      test: ["CMD", "redis-cli", "--raw", "ping"]

  backend:
    build:
      context: ./backend
      dockerfile: ../infra/docker/backend.Dockerfile
      target: development
      args:
        PYTHON_VERSION: 3.11
    restart: unless-stopped
    ports:
      - "8000:8000"
      - "5678:5678"  # Python debugger
    environment:
      <<: *backend-env
    volumes:
      - ./backend:/app:cached
      - backend_cache:/app/.cache
      - ./.env:/app/.env:ro
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      llama:
        condition: service_started
    command: >
      bash -c "
        poetry install --with dev &&
        poetry run alembic upgrade head &&
        poetry run python scripts/ensure_indexes.py &&
        poetry run uvicorn api.main:app --host 0.0.0.0 --port 8000 --reload --log-config logging.yaml
      "
    healthcheck:
      <<: *healthcheck-defaults
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]

  frontend:
    build:
      context: ./frontend
      dockerfile: ../infra/docker/frontend.Dockerfile
      target: development
      args:
        NODE_VERSION: 20
    restart: unless-stopped
    ports:
      - "3000:3000"
      - "9229:9229"  # Node debugger
    environment:
      VITE_API_URL: ${VITE_API_URL:-http://localhost:8000}
      VITE_WS_URL: ${VITE_WS_URL:-ws://localhost:8000/ws}
      VITE_ENVIRONMENT: development
      NODE_ENV: development
      NODE_OPTIONS: --max_old_space_size=4096
    volumes:
      - ./frontend:/app:cached
      - /app/node_modules
      - /app/.next
      - frontend_cache:/app/.cache
    command: >
      sh -c "
        pnpm install --frozen-lockfile &&
        pnpm dev --host 0.0.0.0 --port 3000
      "
    healthcheck:
      <<: *healthcheck-defaults
      test: ["CMD", "curl", "-f", "http://localhost:3000"]

  llama:
    image: ollama/ollama:latest
    restart: unless-stopped
    ports:
      - "11434:11434"
    volumes:
      - ollama_models:/root/.ollama
      - ./scripts/pull-models.sh:/pull-models.sh:ro
    environment:
      OLLAMA_KEEP_ALIVE: 24h
      OLLAMA_MAX_LOADED_MODELS: 2
      OLLAMA_NUM_PARALLEL: 4
    deploy:
      resources:
        limits:
          memory: 8G
        reservations:
          memory: 6G
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    entrypoint: >
      sh -c "
        ollama serve &
        sleep 5 &&
        sh /pull-models.sh &&
        wait
      "

  qdrant:
    image: qdrant/qdrant:latest
    restart: unless-stopped
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - qdrant_data:/qdrant/storage
      - ./scripts/init-collections.sh:/init-collections.sh:ro
    environment:
      QDRANT__SERVICE__HTTP_PORT: 6333
      QDRANT__SERVICE__GRPC_PORT: 6334
      QDRANT__LOG_LEVEL: INFO
      QDRANT__SERVICE__MAX_REQUEST_SIZE_MB: 50
      QDRANT__STORAGE__STORAGE_PATH: /qdrant/storage
      QDRANT__STORAGE__ON_DISK_PAYLOAD: true
    healthcheck:
      <<: *healthcheck-defaults
      test: ["CMD", "curl", "-f", "http://localhost:6333/readiness"]

  minio:
    image: minio/minio:latest
    restart: unless-stopped
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER:-minioadmin}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD:-minioadmin}
      MINIO_BROWSER_REDIRECT_URL: http://localhost:9001
    volumes:
      - minio_data:/data
      - ./scripts/init-buckets.sh:/docker-entrypoint-initdb.d/init-buckets.sh:ro
    command: server /data --console-address ":9001"
    healthcheck:
      <<: *healthcheck-defaults
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]

  maildev:
    image: maildev/maildev:latest
    restart: unless-stopped
    ports:
      - "1080:1080"
      - "1025:1025"
    environment:
      MAILDEV_WEB_PORT: 1080
      MAILDEV_SMTP_PORT: 1025
      MAILDEV_HIDE_EXTENSIONS: STARTTLS

  # Monitoring Stack
  prometheus:
    image: prom/prometheus:latest
    restart: unless-stopped
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=7d'
      - '--web.enable-lifecycle'

  grafana:
    image: grafana/grafana:latest
    restart: unless-stopped
    ports:
      - "3001:3000"
    environment:
      GF_SECURITY_ADMIN_USER: ${GF_ADMIN_USER:-admin}
      GF_SECURITY_ADMIN_PASSWORD: ${GF_ADMIN_PASSWORD:-admin}
      GF_USERS_ALLOW_SIGN_UP: false
      GF_INSTALL_PLUGINS: grafana-clock-panel,grafana-simple-json-datasource
    volumes:
      - grafana_data:/var/lib/grafana
      - ./monitoring/dashboards:/etc/grafana/provisioning/dashboards:ro
      - ./monitoring/datasources:/etc/grafana/provisioning/datasources:ro

  jaeger:
    image: jaegertracing/all-in-one:latest
    restart: unless-stopped
    ports:
      - "16686:16686"
      - "14268:14268"
      - "4318:4318"
    environment:
      COLLECTOR_ZIPKIN_HOST_PORT: ":9411"
      COLLECTOR_OTLP_ENABLED: true

  # Test Infrastructure
  selenium:
    image: selenium/standalone-chrome:latest
    restart: unless-stopped
    ports:
      - "4444:4444"
      - "7900:7900"  # noVNC
    environment:
      SE_NODE_MAX_SESSIONS: 5
      SE_NODE_SESSION_TIMEOUT: 300
      SE_VNC_PASSWORD: secret
    shm_size: 2gb

volumes:
  postgres_data:
  redis_data:
  backend_cache:
  frontend_cache:
  ollama_models:
  qdrant_data:
  minio_data:
  prometheus_data:
  grafana_data:
  vscode-extensions:
  planolo-node-modules:
  planolo-python-venv:
  planolo-postgres-data:
  planolo-redis-data:

networks:
  default:
    name: planolo-network
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16
```

### Enhanced Makefile with All Commands

```makefile
# Planolo Development Makefile
SHELL := /bin/bash
.PHONY: help dev-setup dev-services dev-status test test-unit test-integration test-e2e lint format clean migrate seed logs shell db-shell redis-cli perf security update docker-build docker-push deploy-staging deploy-prod backup restore

# Default target - show help
help:
	@echo "Planolo Development Commands:"
	@echo "  make dev-setup      - Initial setup after cloning"
	@echo "  make dev-services   - Start all development services"
	@echo "  make dev-status     - Check service health"
	@echo "  make test          - Run all tests"
	@echo "  make lint          - Lint all code"
	@echo "  make format        - Format all code"
	@echo "  make clean         - Clean all artifacts"
	@echo "  make logs [svc=]   - Tail service logs"
	@echo "  make shell [svc=]  - Shell into service"
	@echo "  make perf          - Run performance tests"
	@echo "  make security      - Run security scans"
	@echo ""
	@echo "Database Commands:"
	@echo "  make migrate       - Run database migrations"
	@echo "  make seed          - Seed development data"
	@echo "  make db-shell      - PostgreSQL shell"
	@echo "  make redis-cli     - Redis CLI"
	@echo ""
	@echo "Deployment Commands:"
	@echo "  make docker-build  - Build all images"
	@echo "  make deploy-staging - Deploy to staging"
	@echo "  make backup        - Backup production data"

# Initial setup
dev-setup:
	@echo "🚀 Setting up Planolo development environment..."
	@echo "📦 Installing Python dependencies..."
	cd backend && poetry install --with dev,test,docs
	@echo "📦 Installing Node dependencies..."
	cd frontend && pnpm install --frozen-lockfile
	@echo "🗃️  Running database migrations..."
	cd backend && poetry run alembic upgrade head
	@echo "🌱 Seeding development data..."
	cd backend && poetry run python scripts/seed_data.py
	@echo "🤖 Pulling AI models..."
	docker exec $$(docker ps -qf "name=planolo-llama") ollama pull llama3:8b-instruct-q4_K_M
	docker exec $$(docker ps -qf "name=planolo-llama") ollama pull nomic-embed-text
	@echo "🔧 Generating TypeScript API client..."
	cd frontend && pnpm run generate-api-client
	@echo "🏗️  Creating S3 buckets..."
	docker exec $$(docker ps -qf "name=planolo-minio") mc alias set local http://localhost:9000 minioadmin minioadmin
	docker exec $$(docker ps -qf "name=planolo-minio") mc mb local/planolo-exports || true
	docker exec $$(docker ps -qf "name=planolo-minio") mc mb local/planolo-backups || true
	@echo "📊 Initializing vector collections..."
	cd backend && poetry run python scripts/init_qdrant.py
	@echo "🧪 Running smoke tests..."
	make test-unit
	@echo "✅ Setup complete! Run 'make dev-services' to start."

# Start development services
dev-services:
	@echo "🏃 Starting all services..."
	docker-compose up -d postgres redis llama qdrant maildev minio prometheus grafana jaeger
	@echo "⏳ Waiting for services to be healthy..."
	@timeout 60 bash -c 'until docker-compose ps | grep -E "(postgres|redis)" | grep -v "Exit" | grep -q "Up"; do sleep 1; done'
	@echo "🚀 Starting backend..."
	cd backend && poetry run uvicorn api.main:app --reload --host 0.0.0.0 --port 8000 &
	@echo "🚀 Starting frontend..."
	cd frontend && pnpm dev --host 0.0.0.0 --port 3000 &
	@echo "✅ All services started!"
	@make dev-status

# Check service status
dev-status:
	@echo "📊 Service Status:"
	@echo "Frontend:    http://localhost:3000"
	@echo "Backend API: http://localhost:8000/docs"
	@echo "MailDev:     http://localhost:1080"
	@echo "MinIO:       http://localhost:9001 (minioadmin/minioadmin)"
	@echo "Grafana:     http://localhost:3001 (admin/admin)"
	@echo "Jaeger:      http://localhost:16686"
	@echo ""
	@docker-compose ps

# Run all tests
test: test-unit test-integration test-e2e

test-unit:
	@echo "🧪 Running unit tests..."
	cd backend && poetry run pytest tests/unit -v --cov=api --cov-report=term-missing
	cd sim_core && poetry run pytest -v --hypothesis-show-statistics
	cd frontend && pnpm test:unit

test-integration:
	@echo "🧪 Running integration tests..."
	cd backend && poetry run pytest tests/integration -v

test-e2e:
	@echo "🧪 Running E2E tests..."
	cd frontend && pnpm test:e2e

# Lint code
lint:
	@echo "🔍 Linting Python code..."
	cd backend && poetry run ruff check .
	cd backend && poetry run mypy . --ignore-missing-imports
	cd backend && poetry run bandit -r api/
	@echo "🔍 Linting TypeScript code..."
	cd frontend && pnpm run lint
	cd frontend && pnpm run type-check

# Format code
format:
	@echo "✨ Formatting Python code..."
	cd backend && poetry run black .
	cd backend && poetry run ruff --fix .
	@echo "✨ Formatting TypeScript code..."
	cd frontend && pnpm run format

# Clean artifacts
clean:
	@echo "🧹 Cleaning build artifacts..."
	find . -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null || true
	find . -type d -name ".pytest_cache" -exec rm -rf {} + 2>/dev/null || true
	find . -type d -name ".mypy_cache" -exec rm -rf {} + 2>/dev/null || true
	find . -type d -name "node_modules" -exec rm -rf {} + 2>/dev/null || true
	find . -type d -name ".next" -exec rm -rf {} + 2>/dev/null || true
	find . -type d -name "dist" -exec rm -rf {} + 2>/dev/null || true
	find . -type d -name ".coverage" -exec rm -rf {} + 2>/dev/null || true
	find . -type f -name "*.pyc" -delete 2>/dev/null || true
	@echo "🧹 Cleaning Docker volumes..."
	docker-compose down -v

# Database migrations
migrate:
	cd backend && poetry run alembic upgrade head

migrate-create:
	cd backend && poetry run alembic revision --autogenerate -m "$(msg)"

migrate-rollback:
	cd backend && poetry run alembic downgrade -1

# Seed data
seed:
	cd backend && poetry run python scripts/seed_data.py

# Service logs
logs:
	@if [ -z "$(svc)" ]; then \
		docker-compose logs -f --tail=100; \
	else \
		docker-compose logs -f --tail=100 $(svc); \
	fi

# Shell access
shell:
	@if [ -z "$(svc)" ]; then \
		docker-compose exec backend bash; \
	else \
		docker-compose exec $(svc) sh; \
	fi

# Database shell
db-shell:
	docker-compose exec postgres psql -U planolo -d planolo_dev

# Redis CLI
redis-cli:
	docker-compose exec redis redis-cli

# Performance tests
perf:
	@echo "🏃 Running performance tests..."
	cd backend && poetry run locust -f tests/performance/locustfile.py --headless -u 100 -r 10 -t 60s --host http://localhost:8000

# Security scans
security:
	@echo "🔒 Running security scans..."
	cd backend && poetry run safety check
	cd backend && poetry run bandit -r api/ -ll
	cd frontend && pnpm audit
	docker run --rm -v $(PWD):/src aquasec/trivy fs /src

# Update dependencies
update:
	@echo "📦 Updating dependencies..."
	cd backend && poetry update
	cd frontend && pnpm update --interactive
	cd frontend && pnpm run generate-api-client

# Docker build
docker-build:
	@echo "🏗️  Building Docker images..."
	docker build -f infra/docker/backend.Dockerfile -t planolo/backend:latest ./backend
	docker build -f infra/docker/frontend.Dockerfile -t planolo/frontend:latest ./frontend

# Deploy to staging
deploy-staging:
	@echo "🚀 Deploying to staging..."
	cd infra/terraform/environments/staging && terraform apply -auto-approve
	kubectl --context staging apply -k infra/k8s/overlays/staging

# Backup production
backup:
	@echo "💾 Backing up production data..."
	./scripts/backup-prod.sh

# Restore from backup
restore:
	@echo "♻️  Restoring from backup..."
	./scripts/restore-backup.sh $(backup_id)

# Generate API docs
docs:
	cd backend && poetry run python -m api.main --export-openapi > ../docs/api/openapi.json
	cd docs && npm run build

# Run specific service
run-%:
	docker-compose up -d $*

# Stop specific service
stop-%:
	docker-compose stop $*

# Restart specific service
restart-%:
	docker-compose restart $*

# Health check
health:
	@curl -s http://localhost:8000/health | jq .
	@curl -s http://localhost:3000/api/health | jq .

# Database backup
db-backup:
	docker-compose exec postgres pg_dump -U planolo planolo_dev | gzip > backups/dev-$$(date +%Y%m%d-%H%M%S).sql.gz

# Database restore
db-restore:
	gunzip -c $(file) | docker-compose exec -T postgres psql -U planolo planolo_dev
```

### One-Command Setup Instructions

```bash
# Clone repository
git clone https://github.com/yourusername/planolo.git
cd planolo

# Copy environment template
cp .env.example .env

# Open in VS Code (will auto-prompt for Dev Container)
code .

# If not using VS Code Dev Containers:
docker-compose up -d
make dev-setup
make dev-services

# Verify everything is running
make dev-status

# Access services:
# Frontend:     http://localhost:3000
# Backend API:  http://localhost:8000/docs  
# Email UI:     http://localhost:1080
# S3 UI:        http://localhost:9001
# Monitoring:   http://localhost:3001
# Tracing:      http://localhost:16686

# Run tests
make test

# View logs
make logs

# Database access
make db-shell

# Stop everything
docker-compose down
```

## 10. Footnotes & Sources

All architectural decisions are directly traceable to blueprint requirements:

- **§1**: "Flight-simulate life-scale money moves" → Interactive drag-drop timeline, real-time calculations
- **§2**: "Financial anxiety is universal" → AI explanations, probability bands, stress-testing
- **§3**: "CPF, tax and mortgage logic" → Deterministic engine with Singapore-specific rules
- **§4**: "Month 0-1...Month 4-6" → MoSCoW prioritization matching exact timeline
- **§5**: Technology stack exactly as specified with production-grade enhancements
- **§6**: "Free tier...Pro tier...Plus tier" → Feature gating implementation
- **§7**: "Planolo.com" secured; growth loop via shareable scenarios  
- **§8**: "Front-end hire...ML engineer" → Architecture supports gradual team growth
- **§9**: Risk mitigations enhanced with specific metrics and contingencies
- **§10**: "Founder-plus-one team" → Dev tooling optimized for solo productivity

Every technical choice optimizes for shipping velocity while maintaining production quality.