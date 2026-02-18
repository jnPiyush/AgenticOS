# ADR-27: PawConnect Platform Architecture

**Status**: Proposed  
**Date**: 2026-02-18  
**Architect**: AgentX Solution Architect  
**Epic**: #27  
**Related Documents**: [PRD-27.md](../prd/PRD-27.md) | [UX-27.md](../ux/UX-27.md) | [SPEC-27.md](../specs/SPEC-27.md)

---

## Context

PawConnect is a dog social networking platform that enables dog owners to:
1. Create profiles for their dogs with photos and personality traits
2. Find and schedule playdates using AI-powered matching
3. Discover and book local service providers (groomers, trainers, vets, boarding)
4. Create and attend community events
5. Share photos and interact through a social feed

**Key Requirements:**
- Support 10,000 MAU by Month 6
- AI-powered playdate matching with <5s response time
- 99.5% uptime requirement
- WCAG 2.1 AA accessibility compliance
- GDPR/CCPA compliance
- PCI-DSS compliant payment processing

---

## Decision Drivers

1. **Scalability**: Must handle 10K MAU initially, scaling to 50K+ within 12 months
2. **AI/ML Integration**: Requires real-time AI matching with cost efficiency (<$0.02 per match)
3. **Development Velocity**: 12-week MVP timeline with 3 full-stack engineers + 1 AI specialist
4. **Cloud Strategy**: Company standard mandates Azure infrastructure
5. **Security & Compliance**: GDPR, CCPA, PCI-DSS requirements
6. **Cost Optimization**: $5K/month infrastructure budget at launch

---

## Considered Options

### 1. Frontend Architecture

#### Option A: React SPA with Next.js (SELECTED)
**Pros:**
- Server-side rendering for SEO and performance
- Built-in routing and API routes
- Strong ecosystem and team familiarity
- Excellent TypeScript support
- Image optimization built-in

**Cons:**
- Slightly steeper learning curve than plain React
- Bundle size can grow without careful management

#### Option B: React SPA (Create React App)
**Pros:**
- Simpler setup
- Faster initial development

**Cons:**
- No SSR (poor SEO for public dog profiles)
- Manual routing setup
- No API routes
- Limited performance optimization

#### Option C: Vue.js with Nuxt
**Pros:**
- Similar features to Next.js
- Easier learning curve

**Cons:**
- Team unfamiliar with Vue ecosystem
- Smaller community than React

**Decision**: **Next.js** for SSR, performance, and team expertise.

---

### 2. Backend Architecture

#### Option A: Node.js + Express + PostgreSQL (SELECTED)
**Pros:**
- JavaScript across full stack (developer efficiency)
- Excellent async I/O for real-time features
- Large ecosystem for integrations (Stripe, SendGrid, Azure)
- Fast development with TypeScript
- Strong ORM support (Prisma, TypeORM)

**Cons:**
- Requires careful memory management at scale
- Not ideal for CPU-intensive tasks (mitigated by AI service separation)

#### Option B: Python (FastAPI) + PostgreSQL
**Pros:**
- Excellent for AI/ML integration
- Strong typing with Pydantic
- Great performance

**Cons:**
- Team less familiar with Python for web services
- Separate language from frontend (slower development)

#### Option C: .NET Core + SQL Server
**Pros:**
- Enterprise-grade
- Strong Azure integration
- Excellent performance

**Cons:**
- Team unfamiliar with C#
- Slower development velocity
- Higher infrastructure costs (SQL Server licensing)

**Decision**: **Node.js + Express** for development velocity and full-stack JavaScript efficiency.

---

### 3. Database Architecture

#### Option A: PostgreSQL (Azure Database for PostgreSQL) (SELECTED)
**Pros:**
- Excellent relational data modeling for user/dog profiles
- Strong JSON support for flexible dog attributes
- PostGIS for geospatial queries (location-based matching)
- Full-text search capabilities
- ACID compliance for booking/payment transactions
- Azure managed service (automatic backups, scaling)

**Cons:**
- Requires schema migrations
- Potentially complex queries for social feed

#### Option B: MongoDB (Azure Cosmos DB)
**Pros:**
- Flexible schema for dog profiles
- Horizontal scaling built-in

**Cons:**
- Weak transaction support (critical for bookings/payments)
- No geospatial index comparable to PostGIS
- Higher cost on Azure Cosmos DB
- Team less familiar

#### Option C: Hybrid (PostgreSQL + Redis)
**Pros:**
- Redis for caching and real-time features

**Cons:**
- Adds complexity
- Defer to post-MVP optimization

**Decision**: **PostgreSQL** (Azure Database for PostgreSQL) for relational integrity, geospatial support, and team expertise. Redis caching deferred to post-MVP.

---

### 4. AI/ML Architecture

#### Option A: Azure OpenAI Service (SELECTED)
**Pros:**
- Enterprise-grade SLA and compliance (GDPR, HIPAA)
- Direct Azure integration (same cloud provider)
- Structured JSON output for matching logic
- Cost-effective with GPT-4o-mini (~$0.01-0.02 per match)
- No model training/maintenance overhead
- Fallback to rule-based matching if unavailable

**Cons:**
- Requires prompt engineering
- Token costs scale with usage
- Potential latency (mitigated by caching)

#### Option B: Azure AI Foundry (Microsoft Foundry)
**Pros:**
- Unified platform for model deployment
- Multi-model support (Llama, Mistral, GPT)
- Cost optimization through model selection

**Cons:**
- More complex setup
- Overkill for MVP matching use case

#### Option C: Custom ML Model (scikit-learn/TensorFlow)
**Pros:**
- Full control over matching logic
- No per-request costs after training

**Cons:**
- Requires labeled training data (not available at MVP)
- Model training, deployment, and maintenance overhead
- 12-week timeline insufficient for custom ML

**Decision**: **Azure OpenAI Service** (GPT-4o-mini) for MVP with structured prompts. Transition to custom model post-MVP if usage justifies training investment.

---

### 5. Authentication & Authorization

#### Option A: Azure AD B2C (SELECTED)
**Pros:**
- Enterprise-grade identity service
- Social login support (Google, Facebook, Apple)
- GDPR/CCPA compliant out-of-the-box
- Aligns with company Azure strategy
- Built-in MFA and password policies

**Cons:**
- More complex than Auth0
- Higher learning curve

#### Option B: Auth0
**Pros:**
- Simpler developer experience
- Excellent documentation
- Fast integration

**Cons:**
- Additional vendor dependency
- Not aligned with Azure strategy
- Higher cost at scale

#### Option C: Custom JWT Auth
**Pros:**
- Full control
- No external dependency

**Cons:**
- Security risk (homegrown auth)
- Development time (GDPR compliance, social login)
- Not acceptable for production

**Decision**: **Azure AD B2C** for compliance, security, and Azure alignment.

---

### 6. File Storage (Photos)

#### Option A: Azure Blob Storage (SELECTED)
**Pros:**
- Cost-effective for large media files ($0.018/GB)
- Built-in CDN integration (Azure CDN)
- Automatic image optimization (Azure Media Services)
- Aligns with Azure strategy

**Cons:**
- Requires S3-compatible SDK learning

#### Option B: AWS S3
**Pros:**
- Team familiar with S3
- Mature ecosystem

**Cons:**
- Multi-cloud complexity
- Not aligned with Azure strategy

**Decision**: **Azure Blob Storage** with Azure CDN for photo hosting.

---

### 7. Real-time Messaging

#### Option A: Azure SignalR Service (SELECTED)
**Pros:**
- Managed WebSocket service
- Auto-scaling
- Azure integration
- No server infrastructure management

**Cons:**
- Higher cost than self-hosted WebSocket

#### Option B: Socket.io (Self-hosted)
**Pros:**
- Full control
- Lower cost at small scale

**Cons:**
- Requires WebSocket server management
- Scaling complexity (sticky sessions)
- Not aligned with managed service strategy

**Decision**: **Azure SignalR Service** for real-time messaging (in-app chat) to reduce operational overhead.

---

### 8. Payment Processing

#### Option A: Stripe (SELECTED)
**Pros:**
- Industry-leading developer experience
- PCI-DSS compliant (SAQ-A)
- Excellent documentation and SDKs
- Handles complex flows (refunds, disputes, payouts)
- Service provider marketplace features

**Cons:**
- 2.9% + $0.30 transaction fee

#### Option B: PayPal
**Pros:**
- User familiarity

**Cons:**
- Weaker developer experience
- Complex integration

**Decision**: **Stripe** for payment processing with service provider booking fees.

---

### 9. Email Service

#### Option A: Twilio SendGrid (SELECTED)
**Pros:**
- Reliable deliverability
- Transactional + marketing email support
- Template management
- Analytics

**Cons:**
- Cost scales with volume

#### Option B: Azure Communication Services
**Pros:**
- Azure native

**Cons:**
- Less mature than SendGrid
- Weaker template management

**Decision**: **Twilio SendGrid** for transactional emails (account verification, booking confirmations, event reminders).

---

### 10. Monitoring & Observability

#### Option A: Azure Monitor + Application Insights (SELECTED)
**Pros:**
- Native Azure integration
- Automatic application performance monitoring (APM)
- Log aggregation and query (KQL)
- Alerting and dashboards

**Cons:**
- Learning curve for KQL

#### Option B: Datadog
**Pros:**
- Superior dashboarding
- Multi-cloud support

**Cons:**
- Higher cost
- Not aligned with Azure strategy

**Decision**: **Azure Monitor + Application Insights** for cost and Azure alignment.

---

### 11. CI/CD Pipeline

#### Option A: GitHub Actions (SELECTED)
**Pros:**
- Native GitHub integration
- Free for public repos (2,000 minutes/month private)
- YAML-based workflows
- Large action marketplace

**Cons:**
- Concurrency limits on free tier

#### Option B: Azure DevOps Pipelines
**Pros:**
- Azure native
- Advanced pipeline features

**Cons:**
- Less familiar to team
- More complex setup

**Decision**: **GitHub Actions** for CI/CD with automatic deployment to Azure App Service.

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         USERS (Web Browser)                      │
│                    (React/Next.js PWA)                           │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 │ HTTPS
                 │
┌────────────────▼────────────────────────────────────────────────┐
│                     Azure CDN / Front Door                       │
│              (Static Assets, Image Optimization)                 │
└────────────────┬────────────────────────────────────────────────┘
                 │
      ┌──────────┼──────────┐
      │          │           │
      ▼          ▼           ▼
┌─────────┐ ┌─────────┐ ┌─────────────────┐
│ Next.js │ │  API    │ │  Azure SignalR  │
│   SSR   │ │ Server  │ │  (WebSocket)    │
│ (Azure  │ │(Express)│ │                 │
│  App    │ │ (Azure  │ │  Real-time Chat │
│ Service)│ │  App    │ │                 │
│         │ │ Service)│ │                 │
└────┬────┘ └────┬────┘ └─────────────────┘
     │           │
     │      ┌────┴─────────────────────────┐
     │      │                              │
     ▼      ▼                              ▼
┌─────────────────┐              ┌─────────────────┐
│  Azure Blob     │              │   Azure OpenAI  │
│   Storage       │              │    Service      │
│                 │              │                 │
│  Dog Photos     │              │  AI Matching    │
│  User Avatars   │              │  (GPT-4o-mini)  │
└─────────────────┘              └─────────────────┘
                                          │
     ┌────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────────────┐
│         Azure Database for PostgreSQL               │
│                                                     │
│  Tables: users, dogs, profiles, events, bookings,  │
│          service_providers, messages, posts         │
│                                                     │
│  PostGIS Extension: geospatial queries             │
└─────────────────────────────────────────────────────┘
     │
     │
     ▼
┌─────────────────────────────────────────────────────┐
│      External Services (API Integrations)           │
│                                                     │
│  • Stripe (Payments)                                │
│  • SendGrid (Email)                                 │
│  • Google Maps API (Geocoding/Maps)                 │
│  • Azure AD B2C (Authentication)                    │
└─────────────────────────────────────────────────────┘

Monitoring & Logging:
┌─────────────────────────────────────────────────────┐
│  Azure Monitor + Application Insights               │
│  • Request tracing, error tracking                  │
│  • Performance metrics, custom events               │
└─────────────────────────────────────────────────────┘
```

---

## Data Flow: AI Playdate Matching

```
1. User requests playdate matches for their dog (e.g., "Buddy")
                 │
                 ▼
2. API fetches Buddy's profile from PostgreSQL
   (breed, age, size, energy, temperament, location)
                 │
                 ▼
3. API queries PostgreSQL for candidate dogs within radius
   (PostGIS geospatial query: 10-mile radius)
                 │
                 ▼
4. API constructs matching prompt for Azure OpenAI:
   
   Prompt: "You are a dog playdate compatibility expert. Given the
   following dog profiles, calculate compatibility scores (0-100) based
   on size, age, energy level, and temperament. Return JSON with
   dog_id and match_score for each candidate."
   
   Context: 
   - Source Dog: Buddy (Golden Retriever, 2y, high energy, friendly)
   - Candidates: [Max (Lab, 3y, ...), Luna (Poodle, 1y, ...), ...]
                 │
                 ▼
5. Azure OpenAI returns structured JSON:
   {
     "matches": [
       {"dog_id": 123, "match_score": 95, "reason": "Similar size..."},
       {"dog_id": 456, "match_score": 88, "reason": "Compatible energy..."}
     ]
   }
                 │
                 ▼
6. API sorts by match_score, returns top 5 matches to client
                 │
                 ▼
7. Client displays match cards with scores
```

**Fallback**: If Azure OpenAI fails, use rule-based filtering (size + age + energy level thresholds).

---

## Security Architecture

### Authentication Flow
```
User Login → Azure AD B2C → JWT Token → API validates token → Access granted
```

### Authorization Model (RBAC)
- **Dog Owner**: Can create dog profiles, request playdates, book services, RSVP events
- **Service Provider**: Can create business profile, manage bookings, respond to reviews
- **Event Organizer**: Can create events, manage attendees
- **Admin**: Full platform management

### Data Protection
- **Encryption at Rest**: Azure Blob Storage (AES-256), PostgreSQL TDE (Transparent Data Encryption)
- **Encryption in Transit**: TLS 1.3 for all HTTPS traffic
- **Sensitive Data**: Passwords hashed with bcrypt (salt rounds: 12)
- **PII Handling**: User email, phone, address stored with row-level encryption
- **Payment Data**: Tokenized via Stripe (no card data stored)

### Content Moderation
- **Photo Uploads**: Azure Content Safety API (detect inappropriate content)
- **Text Moderation**: Profanity filter for posts/comments (pre-launch manual review)
- **User Reporting**: Block/report functionality for users and service providers

### Compliance
- **GDPR (EU users)**:
  - User consent for data processing
  - Right to access, rectify, delete data
  - Data export functionality
- **CCPA (CA users)**:
  - Do Not Sell My Personal Information
  - Data disclosure requests
- **PCI-DSS**: Stripe handles all payment data (SAQ-A compliance)

---

## Performance & Scalability

### Performance Targets
| Metric | Target | Strategy |
|--------|--------|----------|
| Page Load Time | <2s | SSR, code splitting, image optimization (Azure CDN) |
| API Response Time | <500ms (p95) | Database indexing, query optimization |
| AI Matching Latency | <5s | Prompt optimization, caching common matches |
| Concurrent Users | 1,000 (launch) | Azure App Service scaling plan (S1) |

### Scalability Strategy
- **Horizontal Scaling**: Azure App Service auto-scaling (CPU >70% → add instance)
- **Database Scaling**: Azure PostgreSQL read replicas for analytics queries
- **CDN**: Azure CDN for static assets and images (cache invalidation on upload)
- **Caching** (Post-MVP): Redis for session storage, API response caching

### Database Optimization
- **Indexes**:
  - `dogs(user_id)`, `dogs(location)` (PostGIS spatial index)
  - `events(date, location)`, `bookings(service_provider_id, status)`
  - `posts(user_id, created_at)`
- **Connection Pooling**: PgBouncer for PostgreSQL (max 100 connections)
- **Partitioning** (Post-MVP): Posts table partitioned by month

---

## Deployment Strategy

### Environments
1. **Development**: Local (Docker Compose)
2. **Staging**: Azure App Service (separate resource group)
3. **Production**: Azure App Service (production resource group)

### Deployment Pipeline (GitHub Actions)
```yaml
on: push to main
  → Run tests (Jest, Playwright)
  → Build Next.js (npm run build)
  → Build API (Docker image)
  → Push Docker image to Azure Container Registry
  → Deploy to Azure App Service (staging)
  → Run smoke tests
  → Manual approval gate
  → Deploy to production (blue-green deployment)
```

### Rollback Strategy
- **Blue-Green Deployment**: Azure App Service deployment slots
- **Rollback**: Switch traffic back to previous slot (instant)
- **Database Migrations**: Backward-compatible changes only (no destructive migrations)

### Infrastructure as Code (IaC)
- **Terraform** for Azure resource provisioning (deferred to post-MVP)
- **Manual provisioning** for MVP (documented in deployment guide)

---

## Cost Estimate (Monthly, 10K MAU)

| Service | Cost | Notes |
|---------|------|-------|
| Azure App Service (S1) | $75 | 2 instances (API + Next.js) |
| Azure Database for PostgreSQL (B2s) | $100 | 2 vCores, 4GB RAM |
| Azure Blob Storage | $20 | 1TB photos (~100GB + 900GB headroom) |
| Azure CDN | $50 | 500GB bandwidth |
| Azure SignalR Service (Free) | $0 | 20 concurrent connections |
| Azure OpenAI (GPT-4o-mini) | $500 | 25K matching requests/month @ $0.02/request |
| Stripe Processing Fees | Variable | 2.9% + $0.30 per transaction |
| SendGrid | $20 | 40K emails/month |
| Google Maps API | $100 | Geocoding + Map displays |
| Azure AD B2C | $0 | First 50K MAU free |
| Azure Monitor | $50 | Log ingestion + retention |
| **Total** | **~$915/month** | Excludes Stripe fees |

**Optimization Opportunities**:
- Reduce AI matching costs by caching common results (target: $300/month)
- Use Azure Reserved Instances for 30% discount (post-MVP)

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Azure OpenAI rate limits exceeded | High | Implement request queuing, fallback to rule-based matching |
| Database connection pool exhausted | High | PgBouncer connection pooling, auto-scaling |
| Photo storage costs exceed budget | Medium | Implement image compression (WebP), lazy loading |
| Stripe webhook failures | High | Retry logic with exponential backoff, dead letter queue |
| GDPR/CCPA non-compliance | Critical | Legal review before launch, user consent flows, data export API |

---

## Future Enhancements (Post-MVP)

1. **Redis Caching**: Reduce database load for social feed
2. **Elasticsearch**: Full-text search for service providers and posts
3. **GraphQL API**: Replace REST for mobile app efficiency
4. **WebRTC Video**: Playdate intro calls
5. **Push Notifications**: Native mobile apps (iOS/Android)
6. **Custom ML Model**: Replace Azure OpenAI if cost/accuracy justifies training

---

## Decision Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-02-18 | Next.js + Node.js + PostgreSQL | Full-stack JavaScript, SSR, relational data integrity |
| 2026-02-18 | Azure OpenAI (GPT-4o-mini) for AI matching | Fast MVP delivery, no training data required, fallback to rule-based |
| 2026-02-18 | Azure AD B2C for authentication | Enterprise compliance (GDPR/CCPA), social login, Azure alignment |
| 2026-02-18 | Stripe for payments | PCI-DSS compliance, marketplace features, excellent DX |
| 2026-02-18 | Azure SignalR for real-time messaging | Managed service, auto-scaling, reduces ops overhead |

---

## Approval

| Role | Name | Status | Date |
|------|------|--------|------|
| Architect | AgentX | Draft | 2026-02-18 |
| Engineering Lead | TBD | Pending | - |
| AI/ML Specialist | TBD | Pending | - |
| Security Lead | TBD | Pending | - |

---

**Last Updated**: 2026-02-18  
**Version**: 1.0  
**Next Steps**: Create Technical Specification (SPEC-27.md) with detailed API contracts, database schema, and deployment guide.
