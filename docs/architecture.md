# Architecture Overview - 24h Tremblant D9

## 📋 Table of Contents
- [System Overview](#system-overview)
- [High-Level Architecture](#high-level-architecture)
- [Application Layers](#application-layers)
- [Module Architecture](#module-architecture)
- [Data Model](#data-model)
- [Integration Architecture](#integration-architecture)
- [Deployment Architecture](#deployment-architecture)
- [Security Architecture](#security-architecture)
- [Performance Architecture](#performance-architecture)
- [Scalability Strategy](#scalability-strategy)

---

## 🎯 System Overview

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

### System Metrics

| Metric | Value |
|--------|-------|
| **Custom Modules** | 20 |
| **Custom Themes** | 2 |
| **Contrib Modules** | 70+ |
| **Configuration Files** | 1000+ YAML |
| **Content Types** | 10 |
| **Paragraph Types** | 30+ |
| **API Integrations** | 7 |

---

## 🏗️ High-Level Architecture

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

---

## 🎨 Application Layers

### 1. Presentation Layer

#### Frontend Theme Structure

```
/docroot/themes/custom/tremblant/
├── sass/                    # SCSS source files
│   ├── base/               # Base styles, resets
│   ├── components/         # Reusable components
│   ├── layouts/            # Page layouts
│   └── styles.scss         # Main entry point
├── js/                     # JavaScript files
│   ├── behaviors/          # Drupal behaviors
│   ├── components/         # UI components
│   └── main.js            # Main JS entry
├── templates/              # Twig templates
│   ├── content/           # Content templates
│   ├── layout/            # Layout templates
│   └── navigation/        # Menu templates
├── images/                # Image assets
├── fonts/                 # Web fonts
└── gulpfile.js           # Build configuration
```

#### Admin Theme

```
/docroot/themes/custom/tremblant_admin_theme/
├── css/                   # Admin-specific styles
├── templates/             # Admin UI templates
└── tremblant_admin_theme.info.yml
```

#### Key Frontend Technologies

| Technology | Version | Purpose |
|------------|---------|---------|
| **Cog Base Theme** | 1.15 | Base theme framework |
| **Sass/SCSS** | - | CSS preprocessing |
| **Gulp** | 3.9.x ⚠️ | Build automation |
| **jQuery** | 3.x | DOM manipulation |
| **Slick Carousel** | 1.8 | Image carousels |
| **Masonry** | 4.2 | Grid layouts |
| **Chosen** | 1.8 | Enhanced selects |

---

### 2. Application Layer

#### Drupal Core Services

```mermaid
graph LR
    A[HTTP Request] --> B[Routing]
    B --> C[Controller]
    C --> D[Entity System]
    C --> E[Form API]
    C --> F[Database Layer]
    D --> G[Render Array]
    E --> G
    F --> G
    G --> H[Theme Layer]
    H --> I[HTML Response]
    
    style C fill:#3498db,color:#fff
    style D fill:#27ae60,color:#fff
    style H fill:#f39c12,color:#fff
```

#### Custom Module Organization

**Core Functionality:**
- `tremblant_core` - Central services, event subscribers, utilities
- `tremblant_admin` - Admin tools, settings, batch operations
- `tremblant_info` - Content components, blocks, reusable elements

**Commerce & Payments:**
- `tremblant_commerce` - E-commerce customizations, checkout flows
- `commerce_payfacto` - PayFacto payment gateway integration
- `tremblant_counter` - Real-time donation counters

**Participant Management:**
- `tremblant_participant` - Individual participant features
- `tremblant_dashboard` - User dashboards and toolboxes
- `tremblant_invitations` - Team invitation system

**Rankings & Gamification:**
- `tremblant_classement` - Leaderboard calculations and display
- `tremblant_badges` - Achievement/badge system
- `tremblant_rewards` - Reward management

**Data & Reporting:**
- `tremblant_report` - Analytics, exports, Google Sheets integration
- `tremblant_archive` - Transaction archiving
- `tremblant_event_archive` - Event-level archiving

**Communication:**
- `tremblant_emails` - Email templates and customization
- `tremblant_diffusion` - Live broadcast displays

**Migration:**
- `import_site_info` - D7 to D9 content migration
- `import_site_participant` - User/profile migration

---

### 3. Service Layer

#### Service Architecture

```php
// Example: Donation Calculator Service
namespace Drupal\tremblant_commerce;

class DonationCalculator {
  
  public function calculateTaxReceipt($amount, $advantage = 0) {
    $eligible_amount = $amount - $advantage;
    $minimum_threshold = 20.00;
    
    return [
      'eligible_amount' => max(0, $eligible_amount),
      'is_eligible' => $eligible_amount >= $minimum_threshold,
      'advantage' => $advantage,
    ];
  }
  
  public function calculateTeamProgress($total, $goal) {
    $percentage = $goal > 0 ? ($total / $goal) * 100 : 0;
    
    return [
      'percentage' => min(100, $percentage),
      'remaining' => max(0, $goal - $total),
      'goal_reached' => $total >= $goal,
    ];
  }
}
```

#### Service Registration

```yaml
# tremblant_commerce.services.yml
services:
  tremblant_commerce.donation_calculator:
    class: Drupal\tremblant_commerce\DonationCalculator
    
  tremblant_commerce.order_manager:
    class: Drupal\tremblant_commerce\OrderManager
    arguments: ['@entity_type.manager', '@logger.factory']
    
  tremblant_core.mailchimp:
    class: Drupal\tremblant_core\Service\MailChimpService
    arguments: ['@mailchimp.api', '@config.factory']
```

---

## 🧩 Module Architecture

### Module Dependency Graph

```mermaid
graph TB
    A[tremblant_core] --> B[tremblant_commerce]
    A --> C[tremblant_participant]
    A --> D[tremblant_classement]
    
    B --> E[commerce_payfacto]
    B --> F[tremblant_counter]
    B --> G[tremblant_archive]
    
    C --> H[tremblant_dashboard]
    C --> I[tremblant_invitations]
    C --> J[tremblant_badges]
    
    D --> K[tremblant_diffusion]
    
    L[tremblant_admin] --> A
    L --> M[tremblant_report]
    L --> N[tremblant_event_archive]
    
    O[tremblant_emails] --> A
    P[tremblant_info] --> A
    
    style A fill:#e74c3c,color:#fff
    style B fill:#3498db,color:#fff
    style C fill:#27ae60,color:#fff
```

### Module Interfaces

#### Event Subscribers

```php
// tremblant_core/src/EventSubscriber/OrderCompleteSubscriber.php
namespace Drupal\tremblant_core\EventSubscriber;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Drupal\commerce_order\Event\OrderEvent;

class OrderCompleteSubscriber implements EventSubscriberInterface {
  
  public static function getSubscribedEvents() {
    return [
      'commerce_order.order.paid' => 'onOrderPaid',
    ];
  }
  
  public function onOrderPaid(OrderEvent $event) {
    $order = $event->getOrder();
    
    // Generate tax receipt if eligible
    if ($this->isEligibleForReceipt($order)) {
      $this->generateTaxReceipt($order);
    }
    
    // Update leaderboard
    $this->updateLeaderboard($order);
    
    // Send confirmation email
    $this->sendConfirmationEmail($order);
  }
}
```

#### Plugin System

```php
// commerce_payfacto/src/Plugin/Commerce/PaymentGateway/PayFacto.php
namespace Drupal\commerce_payfacto\Plugin\Commerce\PaymentGateway;

use Drupal\commerce_payment\Plugin\Commerce\PaymentGateway\OnsitePaymentGatewayBase;

/**
 * @CommercePaymentGateway(
 *   id = "payfacto",
 *   label = "PayFacto",
 *   display_label = "Credit Card",
 *   modes = {
 *     "test" = @Translation("Test"),
 *     "live" = @Translation("Live"),
 *   },
 * )
 */
class PayFacto extends OnsitePaymentGatewayBase {
  
  public function createPayment(PaymentInterface $payment, $capture = TRUE) {
    // Implementation
  }
  
  public function refundPayment(PaymentInterface $payment, Price $amount = NULL) {
    // Implementation
  }
}
```

---

## 💾 Data Model

### Entity Relationship Diagram

```mermaid
erDiagram
    USER ||--o{ PROFILE : has
    USER ||--o{ ORDER : places
    
    PROFILE ||--o{ TEAM_MEMBER : "is member of"
    PROFILE ||--o{ BADGE : earns
    
    TEAM ||--o{ TEAM_MEMBER : contains
    TEAM ||--o{ DONATION : receives
    TEAM ||--|| EVENT : "participates in"
    
    ORDER ||--o{ ORDER_ITEM : contains
    ORDER ||--|| PAYMENT : has
    ORDER ||--o{ TAX_RECEIPT : generates
    
    EVENT ||--o{ TEAM : hosts
    EVENT ||--o{ REWARD : offers
    
    DONATION ||--|| ORDER : "created by"
    DONATION ||--|| TEAM : "for"
    
    USER {
        int uid PK
        string name
        string mail
        datetime created
    }
    
    PROFILE {
        int profile_id PK
        int uid FK
        string first_name
        string last_name
        string phone
    }
    
    TEAM {
        int team_id PK
        string name
        int captain_uid FK
        int event_id FK
        decimal goal
        decimal total
    }
    
    ORDER {
        int order_id PK
        int uid FK
        string type
        string state
        decimal total
        datetime completed
    }
    
    PAYMENT {
        int payment_id PK
        int order_id FK
        string gateway
        string token
        decimal amount
    }
```

### Content Types

#### Team Content Type

**Machine name:** `team`  
**Bundle:** `commerce_order`

| Field | Type | Description |
|-------|------|-------------|
| `title` | string | Team name |
| `field_captain` | entity_reference | Team captain (user) |
| `field_event` | entity_reference | Event reference |
| `field_activity` | taxonomy_term | Activity type (running, cycling) |
| `field_goal` | decimal | Fundraising goal |
| `field_participants` | entity_reference | Team members |
| `field_description` | text_long | Team description |
| `field_image` | image | Team photo |

#### Event Content Type

**Machine name:** `event`

| Field | Type | Description |
|-------|------|-------------|
| `title` | string | Event name (e.g., "24h Tremblant 2026") |
| `field_year` | integer | Event year |
| `field_start_date` | datetime | Event start |
| `field_end_date` | datetime | Event end |
| `field_registration_open` | datetime | Registration opens |
| `field_registration_close` | datetime | Registration closes |
| `field_is_active` | boolean | Active event flag |

#### Donation Order Type

**Machine name:** `donation`  
**Bundle:** `commerce_order`

| Field | Type | Description |
|-------|------|-------------|
| `field_team` | entity_reference | Team receiving donation |
| `field_participant` | entity_reference | Participant receiving donation |
| `field_anonymous` | boolean | Anonymous donor |
| `field_message` | text_long | Donor message |
| `field_tax_receipt_issued` | boolean | Receipt generated |

---

## 🔗 Integration Architecture

### Third-Party Integration Flow

```mermaid
sequenceDiagram
    participant User
    participant Drupal
    participant PayFacto
    participant MailChimp
    participant GoogleSheets
    
    User->>Drupal: Make donation
    Drupal->>PayFacto: Process payment
    PayFacto->>Drupal: Payment confirmation
    Drupal->>User: Thank you page
    
    Drupal->>MailChimp: Add/update subscriber
    MailChimp-->>Drupal: Confirmation
    
    Note over Drupal: Nightly cron job
    Drupal->>GoogleSheets: Export reports
    GoogleSheets-->>Drupal: Success
```

### API Integration Details

#### PayFacto Payment Gateway

**Endpoint:** `https://api.payfacto.com/v2/`  
**Authentication:** API Key + Merchant ID

```php
// Request flow
POST /tokenize
{
  "card_number": "4111111111111111",
  "cvv": "123",
  "expiry": "12/25"
}

// Response
{
  "token": "tok_abc123xyz",
  "last4": "1111",
  "brand": "visa"
}

// Process payment
POST /charge
{
  "token": "tok_abc123xyz",
  "amount": 5000, // cents
  "currency": "CAD",
  "merchant_id": "merchant_xxx"
}
```

#### MailChimp API

**Endpoint:** `https://us1.api.mailchimp.com/3.0/`  
**Authentication:** API Key

```php
// Subscribe user
POST /lists/{list_id}/members
{
  "email_address": "user@example.com",
  "status": "subscribed",
  "merge_fields": {
    "FNAME": "John",
    "LNAME": "Doe",
    "TEAM": "Team A"
  }
}
```

#### Google Sheets Export

**Authentication:** Service Account (JSON key)

```php
// Export donations report
$client = new Google_Client();
$client->setAuthConfig('/path/to/credentials.json');
$service = new Google_Service_Sheets($client);

$values = [
  ['Date', 'Team', 'Amount', 'Donor'],
  ['2026-02-20', 'Team A', '$100', 'John Doe'],
  // ... more rows
];

$body = new Google_Service_Sheets_ValueRange(['values' => $values]);
$service->spreadsheets_values->update(
  $spreadsheetId, 
  'Sheet1!A1', 
  $body, 
  ['valueInputOption' => 'RAW']
);
```

---

## 🚀 Deployment Architecture

### Environment Topology

```mermaid
graph TB
    subgraph "Development"
        A[DDEV Local]
        B[Feature Branches]
    end
    
    subgraph "Pantheon Multidev"
        C[sbox2 Environment]
    end
    
    subgraph "Pantheon Test"
        D[Test Environment]
    end
    
    subgraph "Pantheon Live"
        E[Production]
        F[Global CDN]
    end
    
    subgraph "CI/CD"
        G[Azure Pipelines]
        H[Azure DevOps Git]
    end
    
    A --> B
    B --> H
    H --> G
    G --> C
    G --> D
    G --> E
    E --> F
    
    style A fill:#3498db,color:#fff
    style E fill:#27ae60,color:#fff
    style G fill:#e74c3c,color:#fff
```

### Deployment Pipeline

```yaml
# Simplified azure-pipelines-deploy.yml
trigger:
  branches:
    include: [master, sbox2]

stages:
- stage: Build
  jobs:
  - job: BuildArtifact
    steps:
    - task: UsePhpVersion@0
      inputs:
        versionSpec: '8.3'
    - script: composer install --no-dev --optimize-autoloader
    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: '$(System.DefaultWorkingDirectory)'
        artifact: 'drupal-artifact'

- stage: Deploy
  dependsOn: Build
  jobs:
  - job: DeployToPantheon
    steps:
    - task: DownloadPipelineArtifact@2
    - script: |
        git clone ssh://codeserver.dev.xxx@codeserver.dev.xxx.drush.in:2222/~/repository.git pantheon
        rsync -av --exclude='.git' ./ pantheon/
        cd pantheon
        git add -A
        git commit -m "Deploy from Azure $(Build.SourceVersion)"
        git push origin master
    - script: |
        terminus drush 24h-tremblant.test -- updb -y
        terminus drush 24h-tremblant.test -- cim -y
        terminus drush 24h-tremblant.test -- cr
```

---

## 🔐 Security Architecture

### Security Layers

```mermaid
graph TB
    A[User Request] --> B[Pantheon WAF]
    B --> C[Varnish Cache]
    C --> D[Drupal Security Layer]
    
    D --> E[Authentication]
    D --> F[Authorization - RBAC]
    D --> G[Input Validation]
    D --> H[Output Sanitization]
    
    E --> I[Session Management]
    F --> J[Permission Checks]
    G --> K[Form API Validation]
    H --> L[Twig Auto-escaping]
    
    D --> M[Data Layer]
    M --> N[Database - Parameterized Queries]
    M --> O[Redis - Encrypted Tokens]
    
    style B fill:#e74c3c,color:#fff
    style D fill:#f39c12,color:#fff
    style M fill:#27ae60,color:#fff
```

### Security Controls

| Layer | Control | Implementation |
|-------|---------|----------------|
| **Network** | WAF | Pantheon managed |
| **Network** | DDoS Protection | Pantheon CDN |
| **Application** | CSRF Protection | Drupal Form API tokens |
| **Application** | XSS Prevention | Twig auto-escaping |
| **Application** | SQL Injection | PDO prepared statements |
| **Application** | Session Security | HTTPOnly, Secure cookies |
| **Data** | Encryption at Rest | Pantheon encryption |
| **Data** | Encryption in Transit | TLS 1.2+ |
| **Integration** | API Authentication | API keys, OAuth |
| **Integration** | Webhook Validation | HMAC signatures |

---

## ⚡ Performance Architecture

### Caching Strategy

```mermaid
graph LR
    A[Request] --> B{Varnish Cache?}
    B -->|HIT| C[Return Cached]
    B -->|MISS| D[Drupal]
    
    D --> E{Redis Cache?}
    E -->|HIT| F[Return Cached]
    E -->|MISS| G[Database]
    
    G --> H[Render Array]
    H --> I[Dynamic Page Cache]
    I --> J[Response]
    
    J --> K[Store in Varnish]
    K --> C
    
    style B fill:#27ae60,color:#fff
    style E fill:#3498db,color:#fff
    style I fill:#f39c12,color:#fff
```

### Caching Layers

| Layer | TTL | Content | Bypass |
|-------|-----|---------|--------|
| **Varnish (Edge)** | 1 hour | Anonymous pages | Cookie present |
| **Redis (Object)** | 15 min | Entities, queries | Cache clear |
| **Dynamic Page Cache** | Per request | Personalized content | Admin pages |
| **Render Cache** | Until invalidated | Render arrays | N/A |

### Performance Optimizations

```php
// Lazy loading for leaderboard
function tremblant_classement_leaderboard() {
  $build = [
    '#lazy_builder' => ['tremblant_classement.lazy_builder:renderLeaderboard', []],
    '#create_placeholder' => TRUE,
    '#cache' => [
      'keys' => ['leaderboard'],
      'contexts' => ['url.query_args:page'],
      'tags' => ['leaderboard'],
      'max-age' => 300, // 5 minutes
    ],
  ];
  return $build;
}

// BigPipe for donation counter
$build['counter'] = [
  '#lazy_builder' => ['tremblant_counter.lazy_builder:renderCounter', [$team_id]],
  '#create_placeholder' => TRUE,
];
```

---

## 📈 Scalability Strategy

### Horizontal Scaling

**Pantheon Infrastructure:**
- Multiple PHP application containers
- Load balancer distributes traffic
- Shared Redis for cache
- Shared MariaDB with read replicas

### Database Optimization

```sql
-- Indexed columns for performance
CREATE INDEX idx_team_total ON commerce_order (field_team_target_id, total_price);
CREATE INDEX idx_donation_date ON commerce_order (completed, type);
CREATE INDEX idx_participant_team ON profile (field_team_target_id);

-- Partitioning for archived data
CREATE TABLE commerce_order_archive (
  LIKE commerce_order
) PARTITION BY RANGE (YEAR(completed)) (
  PARTITION p2024 VALUES LESS THAN (2025),
  PARTITION p2025 VALUES LESS THAN (2026),
  PARTITION p2026 VALUES LESS THAN (2027)
);
```

### Performance Monitoring

```yaml
# New Relic configuration
services:
  newrelic:
    class: Drupal\newrelic\NewRelicService
    arguments: ['@config.factory']
  
# Track custom metrics
newrelic_custom_metric('Donation/Count', $donation_count);
newrelic_custom_metric('Leaderboard/QueryTime', $query_time);
```

---

## 🔄 Data Flow Diagrams

### Team Registration Flow

```mermaid
flowchart TD
    A[User: Start Registration] --> B{Logged In?}
    B -->|No| C[Facebook OAuth / Register]
    B -->|Yes| D[Team Creation Form]
    C --> D
    
    D --> E[Validate Team Name]
    E -->|Invalid| D
    E -->|Valid| F[Add Participants]
    
    F --> G[Review & Confirm]
    G --> H[Checkout]
    H --> I[PayFacto Payment]
    
    I -->|Success| J[Create Team Entity]
    I -->|Failure| H
    
    J --> K[Create Profiles]
    K --> L[Send Invitations]
    L --> M[Generate Tax Receipt]
    M --> N[Send Confirmation Email]
    N --> O[Redirect to Dashboard]
    
    style I fill:#f39c12,color:#fff
    style J fill:#27ae60,color:#fff
```

### Donation Processing Flow

```mermaid
flowchart TD
    A[User: Visit Team Page] --> B[Click Donate]
    B --> C[Enter Amount & Info]
    C --> D[PayFacto Tokenization]
    D --> E[Create Order]
    E --> F[Process Payment]
    
    F -->|Success| G[Mark Order Paid]
    F -->|Failure| H[Show Error]
    H --> C
    
    G --> I[Update Team Total]
    I --> J[Update Leaderboard]
    J --> K{Amount >= $20?}
    
    K -->|Yes| L[Generate Tax Receipt]
    K -->|No| M[Skip Receipt]
    
    L --> N[Send Thank You Email]
    M --> N
    N --> O[Update MailChimp]
    O --> P[Show Confirmation]
    
    style F fill:#f39c12,color:#fff
    style G fill:#27ae60,color:#fff
    style L fill:#3498db,color:#fff
```

---

## 🏆 Best Practices

### Code Organization

```
/docroot/modules/custom/tremblant_commerce/
├── src/
│   ├── Controller/          # Route controllers
│   ├── Form/               # Form classes
│   ├── Plugin/             # Plugin implementations
│   ├── Service/            # Business logic services
│   ├── EventSubscriber/    # Event subscribers
│   └── Entity/             # Custom entities
├── templates/              # Twig templates
├── tests/
│   ├── src/
│   │   ├── Unit/          # PHPUnit unit tests
│   │   └── Kernel/        # Kernel tests
│   └── features/          # Behat tests
├── tremblant_commerce.info.yml
├── tremblant_commerce.module
├── tremblant_commerce.routing.yml
├── tremblant_commerce.services.yml
└── README.md
```

### Dependency Injection

```php
namespace Drupal\tremblant_commerce\Controller;

use Drupal\Core\Controller\ControllerBase;
use Drupal\tremblant_commerce\Service\DonationCalculator;
use Symfony\Component\DependencyInjection\ContainerInterface;

class DonationController extends ControllerBase {
  
  protected $calculator;
  
  public function __construct(DonationCalculator $calculator) {
    $this->calculator = $calculator;
  }
  
  public static function create(ContainerInterface $container) {
    return new static(
      $container->get('tremblant_commerce.donation_calculator')
    );
  }
  
  public function donate() {
    // Use $this->calculator
  }
}
```

---

**Document Version**: 2.0.0  
**Last Updated**: 2026-02-26  
**Maintained By**: GMA-AI-Lab Development Team

---

## 📝 Change Log

| Date | Version | Changes | Author |
|------|---------|---------|--------|
| 2026-02-26 | 2.0.0 | Complete architecture documentation with all sections | Dev Team |
| 2026-02-26 | 1.0.0 | Initial architecture overview | Dev Team |