# JobHunt AI - Technical Architecture

## System Overview

JobHunt AI employs a microservices architecture deployed as a monorepo, with clear separation between frontend (Next.js), backend (ASP.NET Core), and edge workers (Cloudflare). The system is designed for high availability, scalability, and autonomous operation.

## Architecture Principles

- **Separation of Concerns**: Clear boundaries between presentation, business logic, and data layers
- **Event-Driven**: Asynchronous processing for long-running operations
- **Resilience**: Circuit breakers, retries, and fallback mechanisms
- **Observability**: Comprehensive logging, monitoring, and tracing
- **Security First**: Zero-trust architecture with defense in depth
- **Cloud Native**: Container-based, stateless services with horizontal scaling

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         Users                               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   Cloudflare Edge                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │   CDN    │  │   WAF    │  │  Workers │  │   DDoS   │  │
│  │          │  │          │  │          │  │  Protect │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                        │
│  ┌────────────────────────┐  ┌───────────────────────────┐ │
│  │   Next.js 15 Frontend  │  │  ASP.NET Core Backend     │ │
│  │  - React Server Comp.  │  │  - Web API Controllers    │ │
│  │  - App Router          │  │  - SignalR Hubs          │ │
│  │  - Tailwind CSS        │  │  - Background Services   │ │
│  └────────────────────────┘  └───────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Service Layer                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │   Job    │  │    AI    │  │  Email   │  │  Google  │  │
│  │ Crawler  │  │ Service  │  │ Service  │  │   APIs   │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │Stagehand │  │  Mastra  │  │Analytics │  │  Auth    │  │
│  │          │  │Workflows │  │          │  │ Service  │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Data Layer                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │PostgreSQL│  │  Redis   │  │   Blob   │  │  Vector  │  │
│  │          │  │  Cache   │  │ Storage  │  │    DB    │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## Component Architecture

### Frontend - Next.js 15

```
frontend/
├── app/                      # App Router
│   ├── (auth)/              # Auth group
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/         # Protected routes
│   │   ├── overview/
│   │   ├── applications/
│   │   ├── settings/
│   │   └── analytics/
│   ├── api/                 # API routes
│   └── layout.tsx
├── components/
│   ├── ui/                  # Reusable UI components
│   ├── features/            # Feature-specific components
│   └── layouts/
├── lib/
│   ├── api/                 # API client
│   ├── auth/                # Auth utilities
│   └── utils/
├── hooks/                   # Custom React hooks
├── store/                   # State management (Zustand)
└── types/                   # TypeScript definitions
```

**Key Technologies:**
- React Server Components for improved performance
- App Router for file-based routing
- Tailwind CSS for styling
- Zustand for state management
- React Query for data fetching
- Framer Motion for animations
- React Hook Form for forms
- Zod for validation

### Backend - ASP.NET Core

```
backend/
├── JobHuntAI.API/           # Web API project
│   ├── Controllers/
│   ├── Hubs/               # SignalR hubs
│   ├── Middleware/
│   └── Program.cs
├── JobHuntAI.Core/          # Domain layer
│   ├── Entities/
│   ├── Interfaces/
│   ├── Specifications/
│   └── ValueObjects/
├── JobHuntAI.Application/   # Application layer
│   ├── Services/
│   ├── Commands/
│   ├── Queries/
│   ├── Validators/
│   └── Mappings/
├── JobHuntAI.Infrastructure/# Infrastructure layer
│   ├── Data/
│   ├── Repositories/
│   ├── External/           # External service clients
│   └── Identity/
└── JobHuntAI.Worker/        # Background workers
    ├── Jobs/
    ├── Schedulers/
    └── Processors/
```

**Key Technologies:**
- .NET 8
- Entity Framework Core
- MediatR for CQRS
- FluentValidation
- AutoMapper
- Hangfire for background jobs
- SignalR for real-time updates
- Serilog for logging
- Polly for resilience

### Cloudflare Workers

```
workers/
├── auth-worker/             # Authentication edge logic
├── rate-limiter/           # Rate limiting
├── cache-worker/           # Edge caching
├── security-worker/        # Security headers, CORS
└── analytics-worker/       # Edge analytics
```

**Key Capabilities:**
- JWT validation at edge
- Request rate limiting
- Response caching
- Security headers injection
- Geographic routing
- A/B testing
- Request/response transformation

## Service Architecture

### Job Crawler Service

**Components:**
- **Scheduler**: Cron-based job scheduling
- **Crawler Engine**: Stagehand-powered browser automation
- **Parser**: Job data extraction and normalization
- **Deduplicator**: Duplicate detection using MinHash
- **Scorer**: ML-based relevance scoring

**Flow:**
1. Scheduler triggers crawl jobs
2. Crawler fetches job listings
3. Parser extracts structured data
4. Deduplicator filters duplicates
5. Scorer ranks job relevance
6. Results saved to database

### AI Service

**Components:**
- **Prompt Manager**: Template and prompt versioning
- **LLM Gateway**: Multi-provider support (OpenAI, Anthropic)
- **Content Generator**: Cover letters, resumes
- **Quality Scorer**: Output validation
- **Cache Layer**: Response caching

**Flow:**
1. Receive generation request
2. Fetch user context and job details
3. Build optimized prompt
4. Call LLM with retry logic
5. Validate and score output
6. Cache successful responses

### Email Service

**Components:**
- **Template Engine**: Dynamic email templates
- **Queue Manager**: Priority-based sending queue
- **SMTP Client**: Multi-provider support
- **Tracker**: Open and click tracking
- **Classifier**: Response categorization

**Flow:**
1. Application created triggers email
2. Template rendered with personalization
3. Added to send queue with priority
4. SMTP send with retry logic
5. Track delivery and engagement
6. Parse and classify responses

### Workflow Orchestration (Mastra)

**Workflows:**

1. **Job Discovery Workflow**
   - Trigger: Schedule or manual
   - Steps: Crawl → Parse → Score → Store → Notify

2. **Application Workflow**
   - Trigger: New matched job
   - Steps: Generate Cover Letter → Customize Resume → Find Contacts → Send Application → Update Tracker

3. **Follow-up Workflow**
   - Trigger: Time-based after application
   - Steps: Check Status → Generate Follow-up → Send → Update Tracker

4. **Interview Workflow**
   - Trigger: Interview invitation detected
   - Steps: Parse Details → Create Calendar Event → Send Confirmation → Prepare Materials

## Data Architecture

### Primary Database (PostgreSQL)

**Schema Design:**
- Multi-tenant with row-level security
- Partitioning for large tables (applications, jobs)
- Indexes optimized for common queries
- Full-text search for job descriptions

### Caching Strategy (Redis)

**Cache Layers:**
1. **Session Cache**: User sessions and auth tokens
2. **API Cache**: Frequently accessed data
3. **Job Cache**: Recent job listings
4. **AI Cache**: Generated content
5. **Rate Limit Cache**: Request counting

**Cache Policies:**
- TTL-based expiration
- LRU eviction
- Cache-aside pattern
- Write-through for critical data

### Message Queue (Azure Service Bus / RabbitMQ)

**Queues:**
- `job-discovery`: New jobs to process
- `application-queue`: Applications to send
- `email-queue`: Emails to send
- `ai-generation`: AI content requests
- `analytics-events`: User events

**Topics:**
- `user-events`: User activity broadcast
- `system-alerts`: System-wide notifications
- `job-updates`: Job status changes

### Blob Storage

**Containers:**
- `resumes`: User resume files
- `cover-letters`: Generated cover letters
- `email-attachments`: Email attachments
- `analytics-exports`: Report exports
- `backups`: System backups

## Security Architecture

### Authentication & Authorization

**Authentication Flow:**
1. Google OAuth 2.0 for user login
2. JWT tokens with refresh tokens
3. Token validation at edge (Cloudflare Workers)
4. Session management in Redis

**Authorization:**
- Role-based access control (RBAC)
- Resource-based permissions
- API key authentication for services
- Rate limiting per user/IP

### Data Security

**Encryption:**
- TLS 1.3 for all communications
- AES-256 for data at rest
- Column-level encryption for PII
- Encrypted backups

**Data Protection:**
- Input validation and sanitization
- SQL injection prevention
- XSS protection
- CSRF tokens
- Content Security Policy

### Compliance

**GDPR/CCPA:**
- Data minimization
- Purpose limitation
- Right to deletion API
- Data portability exports
- Consent management

**Audit & Monitoring:**
- All actions logged
- Sensitive data access tracked
- Regular security scans
- Penetration testing

## Deployment Architecture

### Infrastructure as Code

```yaml
# docker-compose.yml
version: '3.8'
services:
  frontend:
    build: ./frontend
    environment:
      - NEXT_PUBLIC_API_URL
    ports:
      - "3000:3000"
  
  backend:
    build: ./backend
    environment:
      - ConnectionStrings__DefaultConnection
      - JWT__Secret
    ports:
      - "5000:5000"
  
  postgres:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
```

### CI/CD Pipeline

**Build Pipeline:**
1. Code commit triggers build
2. Run unit tests
3. Run integration tests
4. Security scanning (SAST/DAST)
5. Build Docker images
6. Push to container registry

**Deploy Pipeline:**
1. Deploy to staging
2. Run smoke tests
3. Performance tests
4. Manual approval gate
5. Blue-green deployment to production
6. Health checks
7. Automatic rollback on failure

### Monitoring & Observability

**Application Monitoring:**
- APM with Application Insights / DataDog
- Custom metrics and dashboards
- Real-time alerting
- Performance profiling

**Infrastructure Monitoring:**
- Server metrics (CPU, memory, disk)
- Network monitoring
- Database performance
- Queue depths

**Logging:**
- Centralized logging (ELK stack)
- Structured logging with correlation IDs
- Log retention policies
- Search and analytics

**Distributed Tracing:**
- End-to-end request tracing
- Service dependency mapping
- Latency analysis
- Error tracking

## Scalability Strategy

### Horizontal Scaling

**Auto-scaling Rules:**
- CPU > 70%: Scale out
- Queue depth > 1000: Add workers
- Response time > 2s: Add instances
- Memory > 80%: Scale out

### Performance Optimization

**Frontend:**
- Code splitting
- Lazy loading
- Image optimization
- CDN for static assets
- Service workers for offline

**Backend:**
- Database connection pooling
- Query optimization
- Response compression
- Output caching
- Async/await patterns

**Database:**
- Read replicas
- Connection pooling
- Query optimization
- Indexing strategy
- Partitioning

## Disaster Recovery

### Backup Strategy

**Backup Schedule:**
- Database: Every 6 hours
- Blob storage: Daily incremental
- Configuration: On change
- Secrets: Weekly to secure vault

### Recovery Procedures

**RPO/RTO Targets:**
- RPO (Recovery Point Objective): 6 hours
- RTO (Recovery Time Objective): 4 hours

**Failover Process:**
1. Automatic health checks detect failure
2. Traffic redirected to standby region
3. Database failover to replica
4. Cache warming from backup
5. Service verification
6. User notification

## Development Workflow

### Monorepo Structure

```
jobhunt-ai/
├── .github/                 # GitHub Actions
├── frontend/               # Next.js application
├── backend/                # ASP.NET Core API
├── workers/                # Cloudflare Workers
├── packages/               # Shared packages
│   ├── types/             # TypeScript types
│   ├── utils/             # Shared utilities
│   └── config/            # Shared configuration
├── scripts/                # Build and deploy scripts
├── docker/                 # Docker configurations
├── terraform/              # Infrastructure as code
├── docs/                   # Documentation
├── wrangler.toml          # Cloudflare configuration
├── docker-compose.yml      # Local development
└── package.json           # Root package.json
```

### Development Tools

**Code Quality:**
- ESLint for JavaScript/TypeScript
- StyleCop for C#
- Prettier for formatting
- Husky for pre-commit hooks
- SonarQube for code analysis

**Testing:**
- Jest for frontend unit tests
- xUnit for backend unit tests
- Playwright for E2E tests
- K6 for load testing
- Postman for API testing

## Technology Stack Summary

### Frontend
- **Framework**: Next.js 15
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **State**: Zustand
- **Data Fetching**: React Query
- **Forms**: React Hook Form
- **Validation**: Zod

### Backend
- **Framework**: ASP.NET Core 8
- **Language**: C#
- **ORM**: Entity Framework Core
- **API**: REST + GraphQL
- **Real-time**: SignalR
- **Background Jobs**: Hangfire
- **Caching**: Redis

### Infrastructure
- **Edge**: Cloudflare Workers
- **Database**: PostgreSQL
- **Cache**: Redis
- **Queue**: Azure Service Bus
- **Storage**: Azure Blob Storage
- **Search**: Elasticsearch
- **Monitoring**: Application Insights

### External Services
- **AI**: OpenAI GPT-4 / Anthropic Claude
- **Email**: SendGrid
- **Auth**: Google OAuth
- **Calendar**: Google Calendar API
- **Sheets**: Google Sheets API
- **Analytics**: Mixpanel
- **Error Tracking**: Sentry