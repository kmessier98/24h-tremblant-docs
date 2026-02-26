# Architecture Overview - 24h Tremblant D9

## Table of Contents
- [System Overview](#system-overview)
- [High-Level Architecture](#high-level-architecture)
- [Application Layers](#application-layers)
- [Module Architecture](#module-architecture)
- [Data Model](#data-model)
- [Integration Architecture](#integration-architecture)
- [Deployment Architecture](#deployment-architecture)
- [Security Architecture](#security-architecture)

---

## System Overview

The 24h Tremblant platform is a complex, event-driven Drupal 11 application designed to manage a 24-hour charity fundraising event. The system handles:

- **Team & Participant Management**: Registration, profiles, invitations
- **E-commerce**: Donations, purchases, payment processing
- **Real-time Tracking**: Leaderboards, counters, progress tracking
- **Reporting**: Analytics, exports to Google Sheets
- **Communication**: Emails, notifications, social integration

### Key Characteristics

- **Platform**: Drupal 11 (Composer-based)
- **Architecture**: Monolithic CMS with service-oriented custom modules
- **Hosting**: Pantheon (managed hosting)
- **Deployment**: Immutable infrastructure via CI/CD
- **Scalability**: Horizontal scaling on Pantheon, Redis caching

---

## High-Level Architecture

```mermaid
graph TB
    subgraph "Presentation Layer"
        A[Web Browser]
        B[Mobile Browser]
        C[Live Diffusion Display]
    end
    
    subgraph "CDN & Edge"
        D[Pantheon Global CDN]
        E[Varnish Cache]
    end
    
    subgraph "Application Layer - Drupal 11"
        F[Routing & Controllers]
        G[Custom Modules - 20]
        H[Contrib Modules - 70+]
        I[Drupal Commerce]
        J[Content Management]
        K[User Management]
    end
    
    subgraph "Service Layer"
        L[Payment Service - PayFacto]
        M[Email Service - MailChimp]
        N[Auth Service - Facebook OAuth]
        O[Storage Service - Google Cloud]
        P[Reporting Service - Google Sheets]
        Q[PDF Service - wkhtmltopdf]
    end
    
    subgraph "Data Layer"
        R[MariaDB 10.3]
        S[Redis Cache]
        T[File System - Pantheon]
        U[Config Management - YAML]
    end
    
    subgraph "CI/CD & DevOps"
        V[Azure Pipelines]
        W[Git - Azure DevOps]
        X[DDEV Local]
        Y[Pantheon Environments]
    end
    
    A --> D
    B --> D
    C --> D
    D --> E
    E --> F
    F --> G
    F --> H
    G --> I
    G --> J
    G --> K
    H --> I
    G --> L
    G --> M
    G --> N
    G --> O
    G --> P
    G --> Q
    F --> R
    F --> S
    F --> T
    F --> U
    V --> Y
    W --> V
    X --> W
    Y --> D
    
    style F fill:#0678be,color:#fff
    style G fill:#f39c12,color:#fff
    style R fill:#27ae60,color:#fff
    style V fill:#e74c3c,color:#fff
```

For the complete architecture documentation including Application Layers, Module Architecture, Data Model, Integration Architecture, Deployment Architecture, Security Architecture, Performance Optimization, and more, please see the full document in the repository.

**Document Version**: 1.0  
**Last Updated**: 2026-02-26  
**Maintained By**: GMA-AI-Lab Development Team