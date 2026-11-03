# Solar Platform - Design Specification

## Overview

Solar Platform is a modular, horizontally-scalable system designed for a solar services startup. The platform consists of customer-facing and internal employee mobile applications backed by a centralized Spring Boot API.

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         Mobile Layer                        │
│  ┌──────────────────┐         ┌──────────────────┐         │
│  │  Customer App    │         │  Internal App    │         │
│  │  (React Native)  │         │  (React Native)  │         │
│  └────────┬─────────┘         └────────┬─────────┘         │
│           │                            │                    │
└───────────┼────────────────────────────┼────────────────────┘
            │                            │
            └────────────┬───────────────┘
                         │ HTTP/REST
┌────────────────────────┼────────────────────────────────────┐
│                    API Gateway Layer                         │
│              Spring Boot 4.1 (Java 21)                       │
│  ┌──────────────────────────────────────────────────────┐   │
│  │            Modular Monolith Architecture             │   │
│  └──────────────────────────────────────────────────────┘   │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────┼────────────────────────────────────┐
│                   Data Layer                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ PostgreSQL   │  │    Redis     │  │ TimescaleDB  │      │
│  │   + Timescale│  │    Cache     │  │  (Time-Series)│      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Backend | Java | 21 |
| Backend Framework | Spring Boot | 4.1 |
| Security | Spring Security | 7 |
| Database | PostgreSQL | 16 |
| Time-Series DB | TimescaleDB | Latest |
| Cache | Redis | 7 |
| Mobile Framework | React Native | 0.86.2 |
| Mobile Runtime | Expo | 57 |
| Frontend Library | React | 19.2.3 |
| Navigation | React Navigation | 6.x |
| Build Tool | Gradle (Kotlin DSL) | 8 |
| Migration Tool | Flyway | 11 |
| API Documentation | SpringDoc OpenAPI | 3.1 |
| Containerization | Docker Compose | Latest |

## Module Architecture

### Modular Monolith Design

The backend follows a modular monolith pattern, where the application is split into logical modules that can be developed, tested, and deployed independently while running in a single JVM.

### Module Breakdown

| Module | Responsibility | Key Features |
|--------|---------------|--------------|
| `solar-app` | Application entry point | Module assembly, Spring Boot configuration |
| `solar-common` | Shared utilities | DTOs, base entities, exceptions, common utilities |
| `solar-auth` | Authentication & Authorization | JWT tokens, Spring Security, OTP management |
| `solar-employee` | Employee management | Profiles, roles, RBAC (Tele-Caller, Sales, Ops, Admin) |
| `solar-customer` | Customer management | Profiles, properties, account lifecycle |
| `solar-onboarding` | Lead-to-customer workflow | 4-phase onboarding: Lead Gen → Tele-Call → Site Meeting → Installation |
| `solar-installation` | Installation management | Installations, components, lifecycle, inverter credentials |
| `solar-inverter` | Inverter integration | Brand adapters (Growatt), metric polling, TimescaleDB ingestion |
| `solar-dispute` | Dispute management | Support tickets, dispute resolution workflow |
| `solar-service` | Service management | Service requests, scheduling, tracking |
| `solar-notification` | Notification system | Push notifications (FCM), SMS, Email |
| `solar-analytics` | Analytics & reporting | Aggregated stats, dashboards, employee performance |

### Module Dependencies

```
solar-app
├── solar-common (shared by all modules)
├── solar-auth
├── solar-employee
├── solar-customer
├── solar-onboarding
├── solar-installation
├── solar-inverter
├── solar-dispute
├── solar-service
├── solar-notification
└── solar-analytics
```

## Data Architecture

### Database Schema Design

#### PostgreSQL (Relational Data)
- **Employee Data**: Profiles, roles, permissions, assignments
- **Customer Data**: Profiles, properties, contact information
- **Lead Data**: Lead generation, qualification, conversion tracking
- **Installation Data**: System configurations, component inventory
- **Service Data**: Requests, schedules, technician assignments
- **Dispute Data**: Tickets, resolutions, communication logs

#### TimescaleDB (Time-Series Data)
- **Inverter Metrics**: Power generation, voltage, current, temperature
- **Performance Data**: Daily/monthly production, efficiency metrics
- **System Health**: Component status, error logs, maintenance alerts

#### Redis (Cache & Session)
- **Session Management**: JWT token storage, refresh tokens
- **API Caching**: Frequently accessed data, computed results
- **Rate Limiting**: Request throttling, abuse prevention
- **Lock Management**: Distributed locks for concurrent operations

### Migration Strategy

- **Tool**: Flyway 11
- **Strategy**: Version-controlled SQL migrations
- **Environment**: Separate migration schemas for dev/staging/prod
- **Rollback**: Supported with down migration scripts

## API Design

### RESTful API Principles

- **Resource-Based URLs**: Noun-based endpoints (e.g., `/api/customers`, `/api/installations`)
- **HTTP Methods**: Proper use of GET, POST, PUT, PATCH, DELETE
- **Status Codes**: Standard HTTP status codes for responses
- **Content Negotiation**: JSON request/response format
- **Versioning**: URL-based versioning (e.g., `/api/v1/`)

### Authentication Flow

```
1. User requests OTP (phone number)
   ↓
2. System generates and sends OTP
   ↓
3. User submits OTP
   ↓
4. System validates OTP
   ↓
5. System issues JWT access token + refresh token
   ↓
6. Client includes JWT in Authorization header
   ↓
7. System validates JWT on each request
   ↓
8. Refresh token used to obtain new access token
```

### API Documentation

- **Tool**: SpringDoc OpenAPI 3.1
- **UI**: Swagger UI at `/swagger-ui/index.html`
- **Format**: OpenAPI 3.0 specification
- **Auto-Generation**: Annotations-based documentation generation

## Mobile Application Architecture

### Customer App

**Purpose**: Solar system monitoring and support requests

**Key Features**:
- Real-time system performance monitoring
- Historical production data visualization
- Service request submission
- Dispute ticket creation
- Profile management

**Tech Stack**:
- React 19.2.3
- React Native 0.86.2
- Expo 57
- React Navigation 6.x

### Internal App

**Purpose**: Employee management and operations

**Key Features**:
- Role-based dashboards (Tele-Caller, Sales, Operations, Admin)
- Lead management and qualification
- Site meeting scheduling
- Installation tracking
- Service request assignment
- Customer communication

**Tech Stack**:
- React 19.2.3
- React Native 0.86.2
- Expo 57
- React Navigation 6.x
- Location Services
- Document Picker
- Image Picker

### Shared Package

**Purpose**: Common utilities and API client

**Contents**:
- API client with authentication
- Shared type definitions
- Common UI components
- Utility functions
- Configuration management

## Security Architecture

### Authentication & Authorization

**Authentication**:
- JWT-based stateless authentication
- OTP-based initial authentication
- Refresh token rotation
- Token expiration management

**Authorization**:
- Role-Based Access Control (RBAC)
- Permission-based endpoint protection
- Employee role hierarchy
- Customer data isolation

### Security Measures

- **Input Validation**: Request validation on all endpoints
- **SQL Injection Prevention**: Parameterized queries
- **XSS Protection**: Input sanitization
- **CSRF Protection**: Token-based CSRF protection
- **Rate Limiting**: Request throttling per user/IP
- **Secure Headers**: Security headers configuration
- **Encryption**: TLS for data in transit
- **Password Hashing**: BCrypt for sensitive data

## Deployment Architecture

### Local Development

```
Docker Compose
├── PostgreSQL 16 + TimescaleDB (Port 5435)
└── Redis 7 (Port 6381)
```

### Production Considerations

**Scalability**:
- Horizontal scaling of Spring Boot instances
- Database read replicas for query scaling
- Redis cluster for distributed caching
- Load balancer for API distribution

**Monitoring**:
- Spring Boot Actuator endpoints
- Application health checks
- Performance metrics collection
- Error tracking and logging

**Backup & Recovery**:
- PostgreSQL regular backups
- TimescaleDB data retention policies
- Redis persistence configuration
- Disaster recovery procedures

## Development Workflow

### Module Creation Process

1. Create module directory `server/solar-<name>/`
2. Add `build.gradle.kts` with module configuration
3. Create source structure `src/main/java/`
4. Add module to `settings.gradle.kts`
5. Add dependency in `solar-app/build.gradle.kts`
6. Spring component scanning auto-registers beans

### Module Removal Process

1. Remove dependency from `solar-app/build.gradle.kts`
2. Remove module from `settings.gradle.kts`
3. Delete module directory (optional)

### Code Organization

```
server/
├── solar-app/              # Main application
├── solar-common/           # Shared code
├── solar-auth/             # Authentication
├── solar-employee/         # Employee management
├── solar-customer/         # Customer management
├── solar-onboarding/       # Onboarding workflow
├── solar-installation/     # Installation management
├── solar-inverter/         # Inverter integration
├── solar-dispute/          # Dispute management
├── solar-service/          # Service management
├── solar-notification/     # Notifications
└── solar-analytics/        # Analytics

mobile/
├── packages/
│   ├── customer-app/       # Customer mobile app
│   ├── internal-app/       # Internal mobile app
│   └── shared/             # Shared utilities
```

## Performance Considerations

### Caching Strategy

- **Read-Through Cache**: Frequently accessed data
- **Write-Through Cache**: Data consistency
- **Cache Invalidation**: Time-based and event-based
- **Distributed Cache**: Redis for multi-instance scenarios

### Database Optimization

- **Indexing**: Strategic indexes on query-heavy tables
- **Query Optimization**: N+1 query prevention
- **Connection Pooling**: HikariCP configuration
- **Batch Operations**: Bulk insert/update operations

### Time-Series Optimization

- **Hypertable Partitioning**: Automatic time-based partitioning
- **Continuous Aggregates**: Pre-computed rollups
- **Retention Policies**: Automatic data cleanup
- **Compression**: TimescaleDB compression policies

## Future Enhancements

### Planned Features

- **Analytics Module**: Advanced dashboards and reporting
- **Real-time Notifications**: WebSocket-based updates
- **Predictive Maintenance**: ML-based failure prediction
- **Energy Trading**: Peer-to-peer energy marketplace
- **Mobile Offline Support**: Offline-first architecture
- **Multi-tenancy**: Support for multiple solar service providers

### Scalability Improvements

- **Microservices Migration**: Gradual extraction to microservices
- **Event-Driven Architecture**: Message queue integration
- **API Gateway**: Centralized API management
- **Service Mesh**: Inter-service communication management

## Contributing Guidelines

### Code Standards

- **Java**: Follow Google Java Style Guide
- **JavaScript/TypeScript**: ESLint + Prettier configuration
- **Documentation**: Javadoc for Java, JSDoc for JavaScript
- **Testing**: Unit tests for business logic, integration tests for APIs

### Git Workflow

- **Branch Strategy**: Feature branches from main
- **Commit Messages**: Conventional Commits specification
- **Pull Requests**: Code review required before merge
- **CI/CD**: Automated testing on pull requests
