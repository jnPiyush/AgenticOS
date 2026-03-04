# PawConnect - Dog Social Network Platform 🐕

[![Status](https://img.shields.io/badge/Status-Architecture%20Complete-blue)]()
[![Epic](https://img.shields.io/badge/Epic-%2327-purple)]()
[![Priority](https://img.shields.io/badge/Priority-P1-orange)]()

> A comprehensive social networking platform for dog owners - enabling community connection, service discovery, event participation, and intelligent playdate coordination.

---

## 📋 Project Overview

**Vision**: Facebook for dogs with AI-powered playdate matching

PawConnect solves a critical problem: dog owners struggle to find compatible playmates, discover reliable services, and connect with local communities. Current solutions are fragmented across multiple platforms (NextDoor, Facebook Groups, Rover, Yelp).

### Key Features
- 🐶 **Dog Profile Management**: Detailed profiles with photos and personality traits
- 🤝 **AI Playdate Matching**: Intelligent compatibility scoring using Azure OpenAI
- 🛠️ **Service Directory**: Vetted groomers, trainers, vets, boarding facilities
- 📅 **Community Events**: Meetups, training classes, adoption drives
- 💬 **Social Feed**: Share photos, like, comment, follow other dog owners

### Success Metrics (6-month targets)
- 10,000 Monthly Active Users
- 500 Weekly Playdates Scheduled
- 60% Event Attendance Rate
- 40% D30 User Retention

---

## 📚 Documentation

### Phase 1: Discovery & Design ✅
- **[PRD-27.md](docs/prd/PRD-27.md)** - Product Requirements Document
  - Problem statement, user personas, features, user stories, timeline
  - 20 user stories across 5 features (P0 and P1 priorities)
- **[UX-27.md](docs/ux/UX-27.md)** - UX Design Specification
  - User flows, wireframes, design system, accessibility
  - Interactive HTML prototypes in `docs/ux/prototypes/`

### Phase 2: Architecture ✅
- **[ADR-27.md](docs/adr/ADR-27.md)** - Architecture Decision Record
  - Technology stack selection rationale
  - AI/ML architecture (Azure OpenAI for matching)
  - Infrastructure decisions (Azure cloud, PostgreSQL, Next.js)
  - Security, scalability, and cost considerations
- **[SPEC-27.md](docs/specs/SPEC-27.md)** - Technical Specification
  - System architecture diagrams
  - Complete database schema (Prisma + PostGIS)
  - REST API specification (40+ endpoints)
  - AI matching service implementation details
  - Deployment guide and monitoring strategy

---

## 🛠️ Technology Stack

### Frontend
- **Framework**: Next.js 14 (React 18 + TypeScript)
- **Styling**: Tailwind CSS
- **State**: React Query + Context API
- **Maps**: Leaflet.js + Google Maps API

### Backend
- **Runtime**: Node.js 20 + Express.js (TypeScript)
- **Database**: PostgreSQL 16 + PostGIS (Azure Database)
- **ORM**: Prisma 5.x
- **AI/ML**: Azure OpenAI (GPT-4o-mini)

### Infrastructure (Azure)
- **Hosting**: Azure App Service
- **Storage**: Azure Blob Storage + CDN
- **Auth**: Azure AD B2C
- **Messaging**: Azure SignalR Service
- **Monitoring**: Azure Monitor + Application Insights

### External Services
- **Payments**: Stripe
- **Email**: Twilio SendGrid
- **Geocoding**: Google Maps API
- **Content Moderation**: Azure Content Safety API

---

## 🏗️ Project Structure

```
pawconnect/
├── docs/
│   ├── prd/
│   │   └── PRD-27.md           # Product Requirements
│   ├── ux/
│   │   ├── UX-27.md            # UX Specification
│   │   └── prototypes/         # Interactive HTML/CSS prototypes
│   │       ├── index.html      # Landing & Feed
│   │       ├── matching.html   # Playdate Matching
│   │       ├── profile.html    # Dog Profile
│   │       ├── services.html   # Service Directory
│   │       └── events.html     # Events Calendar
│   ├── adr/
│   │   └── ADR-27.md           # Architecture Decisions
│   └── specs/
│       └── SPEC-27.md          # Technical Specification
├── frontend/                   # (To be implemented)
│   ├── pages/                  # Next.js pages
│   ├── components/             # React components
│   ├── lib/                    # Utilities
│   └── styles/                 # CSS/Tailwind
├── api/                        # (To be implemented)
│   ├── src/
│   │   ├── routes/             # Express routes
│   │   ├── services/           # Business logic
│   │   ├── models/             # Prisma models
│   │   └── middleware/         # Auth, validation
│   └── prisma/
│       └── schema.prisma       # Database schema
└── README.md                   # This file
```

---

## 🚀 Next Steps

### Phase 3: Implementation (Week 1-12)
1. **Week 1-4: Foundation**
   - Set up repository structure (monorepo: frontend + api)
   - Configure Azure resources (App Service, PostgreSQL, Blob Storage)
   - Implement authentication (Azure AD B2C integration)
   - Database setup (Prisma migrations, PostGIS)
   - Dog profile CRUD with photo uploads

2. **Week 5-8: Core Features**
   - Service provider directory and search
   - Booking request system
   - Event creation and RSVP
   - Reviews and ratings

3. **Week 9-11: AI & Social**
   - AI-powered playdate matching (Azure OpenAI)
   - Playdate scheduling and safety checklist
   - Social feed (posts, likes, comments, follows)
   - Real-time messaging (Azure SignalR)

4. **Week 12: Launch Prep**
   - Performance optimization and load testing
   - Security audit (Azure Security Center)
   - Beta program launch (100 users)
   - Deployment automation (GitHub Actions)

### Phase 4: Launch & Growth
- Public launch (Month 4)
- Feature iteration based on user feedback
- Scale infrastructure as user base grows

---

## 📊 System Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Users (Web Browser)                      │
│                    Next.js PWA (React)                       │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            │ REST API + WebSocket
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                    Azure Front Door (CDN)                    │
└───────────────────────────┬─────────────────────────────────┘
                            │
                ┌───────────┴──────────┐
                │                      │
                ▼                      ▼
┌──────────────────────┐   ┌──────────────────────┐
│   Next.js Server     │   │   Express.js API     │
│   (SSR + API Routes) │   │   (Business Logic)   │
└──────────────────────┘   └──────┬───────────────┘
                                   │
              ┌────────────────────┼────────────────┐
              │                    │                │
              ▼                    ▼                ▼
┌──────────────────┐   ┌──────────────────┐   ┌──────────────┐
│   PostgreSQL     │   │  Azure OpenAI    │   │ Azure Blob   │
│   + PostGIS      │   │  (AI Matching)   │   │   Storage    │
│  (User/Dog Data) │   │                  │   │   (Photos)   │
└──────────────────┘   └──────────────────┘   └──────────────┘
```

---

## 🔒 Security & Compliance

- **Authentication**: Azure AD B2C (OAuth 2.0, social login)
- **Authorization**: RBAC (owner, service provider, admin)
- **Data Protection**: TLS 1.3 in transit, AES-256 at rest
- **GDPR/CCPA**: User consent, data export, right to deletion
- **PCI-DSS**: Stripe payment processing (SAQ-A compliance)
- **Content Moderation**: Azure Content Safety API for photos

---

## 👥 Team & Roles

- **Product Manager**: PRD, feature prioritization, success metrics
- **UX Designer**: User flows, wireframes, prototypes, design system
- **Solution Architect**: Technology decisions, system design, deployment strategy
- **Full-Stack Engineers (3)**: Implementation (frontend + backend)
- **AI/ML Specialist**: Azure OpenAI integration, matching algorithm
- **QA Engineer**: Test strategy, E2E testing, performance testing

---

## 📝 Development Workflow

### Branching Strategy
- `main`: Production-ready code
- `develop`: Integration branch for features
- `feature/*`: Feature branches (e.g., `feature/playdate-matching`)
- `bugfix/*`: Bug fix branches

### CI/CD Pipeline (GitHub Actions)
1. Pull Request → Run tests (Jest, Playwright)
2. Merge to `main` → Build & deploy to staging
3. Manual approval → Deploy to production (blue-green)

### Testing Requirements
- **Unit Tests**: 80% coverage (Jest)
- **Integration Tests**: API endpoints (Supertest)
- **E2E Tests**: Critical user flows (Playwright)
- **Performance Tests**: Load testing (k6, 500 concurrent users)

---

## 💰 Cost Estimate (10K MAU)

| Service | Monthly Cost |
|---------|-------------|
| Azure App Service (S1) | $75 |
| PostgreSQL (B2s) | $100 |
| Blob Storage + CDN | $70 |
| Azure OpenAI (matching) | $500 |
| Azure SignalR (Free tier) | $0 |
| Azure AD B2C | $0 (first 50K MAU) |
| Stripe fees | Variable (2.9% + $0.30) |
| SendGrid | $20 |
| Google Maps API | $100 |
| Azure Monitor | $50 |
| **Total** | **~$915/month** |

---

## 📖 References

- [Product Requirements](docs/prd/PRD-27.md) - Full feature specifications
- [Architecture Decisions](docs/adr/ADR-27.md) - Technology rationale
- [Technical Spec](docs/specs/SPEC-27.md) - Implementation details
- [UX Design](docs/ux/UX-27.md) - Design guidelines

---

## 📄 License

Copyright © 2026 PawConnect. All rights reserved.

---

**Status**: Architecture phase complete ✅  
**Next Milestone**: Development sprint kickoff  
**Target Launch**: Q2 2026 (Week 13)
