# API Integrations Guide - 24h Tremblant D9

## 📋 Table of Contents

- [Integration Overview](#integration-overview)
- [PayFacto Payment Gateway](#payfacto-payment-gateway)
- [MailChimp Email Marketing](#mailchimp-email-marketing)
- [Facebook OAuth Authentication](#facebook-oauth-authentication)
- [Google Cloud Storage](#google-cloud-storage)
- [Google Sheets Reporting](#google-sheets-reporting)
- [reCAPTCHA v3](#recaptcha-v3)
- [Google Tag Manager](#google-tag-manager)
- [Integration Best Practices](#integration-best-practices)
- [Troubleshooting](#troubleshooting)

---

## 🎯 Integration Overview

The 24h Tremblant platform integrates with 7 third-party services to provide comprehensive functionality for the charity event.

### Integration Summary

| Service | Purpose | Status | Critical | Documentation |
|---------|---------|--------|----------|---------------|
| **PayFacto** | Payment processing | ✅ Active | 🔴 HIGH | [↓](#payfacto-payment-gateway) |
| **MailChimp** | Email marketing | ✅ Active | 🟡 MEDIUM | [↓](#mailchimp-email-marketing) |
| **Facebook OAuth** | Social authentication | ✅ Active | 🟢 LOW | [↓](#facebook-oauth-authentication) |
| **Google Cloud Storage** | File storage | ⚠️ Configured | 🟡 MEDIUM | [↓](#google-cloud-storage) |
| **Google Sheets** | Reporting | ✅ Active | 🟢 LOW | [↓](#google-sheets-reporting) |
| **reCAPTCHA v3** | Spam protection | ✅ Active | 🟡 MEDIUM | [↓](#recaptcha-v3) |
| **Google Tag Manager** | Analytics | ✅ Active | 🟢 LOW | [↓](#google-tag-manager) |

### Integration Architecture

```mermaid
graph TB
    A[Drupal Application] --> B[PayFacto API]
    A --> C[MailChimp API]
    A --> D[Facebook OAuth]
    A --> E[Google Cloud Storage]
    A --> F[Google Sheets API]
    A --> G[reCAPTCHA v3 API]
    A --> H[Google Tag Manager]
    
    B --> I[Payment Processing]
    C --> J[Email Campaigns]
    D --> K[User Authentication]
    E --> L[File Storage]
    F --> M[Report Exports]
    G --> N[Bot Protection]
    H --> O[Analytics Tracking]
    
    style B fill:#e74c3c,color:#fff
    style C fill:#3498db,color:#fff
    style E fill:#27ae60,color:#fff
```

---

## 💳 PayFacto Payment Gateway

### Overview

PayFacto is a Canadian payment processor specializing in charity and non-profit organizations. It handles all payment processing for the 24h Tremblant platform.

**Module:** `commerce_payfacto` (custom)  
**Integration Type:** REST API  
**PCI-DSS Compliance:** Level 2

### Key Features

- Credit card processing (Visa, Mastercard, Amex)
- Tokenization (PCI-compliant)
- Recurring payments support
- Canadian market focus
- Real-time transaction processing
- Webhook notifications
- Refund support

### Configuration

#### API Credentials

```yaml
# Configuration stored in: commerce_payment.commerce_payment_gateway.payfacto

# Test Mode
test:
  merchant_id: "test_merchant_12345"
  api_key: "test_api_key_abcdef"
  endpoint: "https://sandbox.payfacto.com/api/v2"

# Live Mode
live:
  merchant_id: "prod_merchant_67890"
  api_key: "prod_api_key_xyz123"
  endpoint: "https://api.payfacto.com/api/v2"
```

#### Drupal Configuration

```php
// docroot/sites/default/settings.php
// Use environment variables for security
$config['commerce_payment.commerce_payment_gateway.payfacto']['configuration'] = [
  'merchant_id' => getenv('PAYFACTO_MERCHANT_ID'),
  'api_key' => getenv('PAYFACTO_API_KEY'),
  'mode' => getenv('PAYFACTO_MODE') ?: 'test',
  'webhook_secret' => getenv('PAYFACTO_WEBHOOK_SECRET'),
];
```

### Integration Flow

```mermaid
sequenceDiagram
    participant User
    participant Drupal
    participant PayFacto
    participant Bank

    User->>Drupal: Enter payment details
    Drupal->>PayFacto: POST /tokenize (card data)
    PayFacto->>Drupal: Return token
    Note over Drupal: Store token only
    
    Drupal->>PayFacto: POST /charge (token + amount)
    PayFacto->>Bank: Authorize transaction
    Bank->>PayFacto: Authorization response
    PayFacto->>Drupal: Payment result
    
    alt Payment Success
        Drupal->>User: Order confirmation
        PayFacto->>Drupal: Webhook notification
        Drupal->>User: Tax receipt (if eligible)
    else Payment Failure
        Drupal->>User: Error message
    end
```

### API Endpoints

#### 1. Tokenize Card

```http
POST https://api.payfacto.com/api/v2/tokenize
Content-Type: application/json
Authorization: Bearer {API_KEY}

{
  "merchant_id": "merchant_xxx",
  "card_number": "4111111111111111",
  "cvv": "123",
  "expiry_month": "12",
  "expiry_year": "2025",
  "cardholder_name": "John Doe"
}
```

**Response:**
```json
{
  "success": true,
  "token": "tok_abc123xyz",
  "last4": "1111",
  "brand": "visa",
  "expiry": "12/25"
}
```

#### 2. Process Payment

```http
POST https://api.payfacto.com/api/v2/charge
Content-Type: application/json
Authorization: Bearer {API_KEY}

{
  "merchant_id": "merchant_xxx",
  "token": "tok_abc123xyz",
  "amount": 10000,  // cents
  "currency": "CAD",
  "description": "Donation to Team A",
  "metadata": {
    "order_id": "12345",
    "team_id": "67",
    "donor_email": "john@example.com"
  }
}
```

**Response:**
```json
{
  "success": true,
  "transaction_id": "txn_def456ghi",
  "status": "completed",
  "amount": 10000,
  "currency": "CAD",
  "authorization_code": "AUTH123"
}
```

#### 3. Refund Payment

```http
POST https://api.payfacto.com/api/v2/refund
Content-Type: application/json
Authorization: Bearer {API_KEY}

{
  "merchant_id": "merchant_xxx",
  "transaction_id": "txn_def456ghi",
  "amount": 10000,  // optional, defaults to full refund
  "reason": "Customer request"
}
```

### Webhook Integration

#### Webhook Events

| Event | Trigger | Action |
|-------|---------|--------|
| `payment.succeeded` | Payment authorized | Mark order as paid |
| `payment.failed` | Payment declined | Update order status |
| `payment.refunded` | Refund processed | Update order, notify user |
| `payment.disputed` | Chargeback initiated | Flag for review |

#### Webhook Endpoint

**URL:** `https://24htremblant.com/payfacto/webhook`

```php
// commerce_payfacto/src/Controller/WebhookController.php

namespace Drupal\commerce_payfacto\Controller;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class WebhookController extends ControllerBase {
  
  public function webhook(Request $request) {
    // 1. Validate webhook signature
    $payload = $request->getContent();
    $signature = $request->headers->get('X-PayFacto-Signature');
    
    if (!$this->validateSignature($payload, $signature)) {
      \Drupal::logger('commerce_payfacto')->error('Invalid webhook signature');
      return new Response('Invalid signature', 403);
    }
    
    // 2. Parse payload
    $data = json_decode($payload, TRUE);
    $event_type = $data['event'];
    $transaction_id = $data['transaction_id'];
    
    // 3. Process event
    switch ($event_type) {
      case 'payment.succeeded':
        $this->processPaymentSuccess($data);
        break;
        
      case 'payment.failed':
        $this->processPaymentFailure($data);
        break;
        
      case 'payment.refunded':
        $this->processRefund($data);
        break;
    }
    
    return new Response('Webhook processed', 200);
  }
  
  private function validateSignature($payload, $signature) {
    $secret = $this->config('commerce_payfacto.settings')->get('webhook_secret');
    $expected_signature = hash_hmac('sha256', $payload, $secret);
    return hash_equals($expected_signature, $signature);
  }
}
```

### Security Best Practices

```php
// ❌ NEVER DO THIS - Store card data
$payment_data = [
  'card_number' => '4111111111111111',  // VIOLATION
  'cvv' => '123',  // VIOLATION
];

// ✅ CORRECT - Use tokenization
$payment_data = [
  'token' => $encrypted_token,
  'last4' => '1111',  // OK to store
  'brand' => 'visa',  // OK to store
];
```

**PCI-DSS Requirements:**
- Never store CVV/CVC
- Never store full card numbers
- Always use HTTPS
- Validate webhook signatures
- Log all transactions
- Implement rate limiting

### Error Handling

```php
try {
  $result = $payfacto_gateway->createPayment($payment);
  
} catch (PaymentDeclinedException $e) {
  // Card declined
  drupal_set_message('Your payment was declined. Please try another card.', 'error');
  
} catch (PaymentGatewayException $e) {
  // Gateway error
  \Drupal::logger('commerce_payfacto')->error($e->getMessage());
  drupal_set_message('Payment processing error. Please try again.', 'error');
  
} catch (\Exception $e) {
  // Unexpected error
  \Drupal::logger('commerce_payfacto')->critical($e->getMessage());
  drupal_set_message('An unexpected error occurred.', 'error');
}
```

---

## 📧 MailChimp Email Marketing

### Overview

MailChimp handles all email marketing for the 24h Tremblant platform, including newsletters, campaign updates, and donor communications.

**Module:** `mailchimp` (contrib)  
**Integration Type:** REST API v3.0

### Features

- Automatic subscriber sync
- List management
- Campaign tracking
- Merge field mapping
- Webhook integration
- Batch operations

### Configuration

#### API Setup

```yaml
# Configuration: mailchimp.settings

api_key: "xxxxxxxxxx-us1"
list_id: "abc123def456"
endpoint: "https://us1.api.mailchimp.com/3.0/"
```

#### List Configuration

**Main List:** "24h Tremblant Participants"  
**List ID:** `abc123def456`

**Merge Fields:**
| Field | MailChimp Tag | Description |
|-------|---------------|-------------|
| First Name | FNAME | Participant first name |
| Last Name | LNAME | Participant last name |
| Team | TEAM | Team name |
| Role | ROLE | Captain / Participant |
| Event Year | YEAR | Event year (2026) |
| Donation Total | DONATED | Total donated amount |

### Integration Flow

```mermaid
sequenceDiagram
    participant User
    participant Drupal
    participant MailChimp
    
    User->>Drupal: Register / Donate
    Drupal->>MailChimp: Subscribe user
    MailChimp->>Drupal: Confirmation
    
    Note over Drupal: Nightly cron
    Drupal->>MailChimp: Sync updates (batch)
    MailChimp->>Drupal: Success
    
    MailChimp->>Drupal: Webhook (unsubscribe)
    Drupal->>Drupal: Update user preference
```

### API Usage

#### Subscribe User

```php
// docroot/modules/custom/tremblant_core/src/Service/MailChimpService.php

namespace Drupal\tremblant_core\Service;

class MailChimpService {
  
  public function subscribeUser($email, $first_name, $last_name, $team_name = NULL) {
    $mailchimp = \Drupal::service('mailchimp.api');
    $list_id = $this->config->get('list_id');
    
    $member_info = [
      'email_address' => $email,
      'status' => 'subscribed',
      'merge_fields' => [
        'FNAME' => $first_name,
        'LNAME' => $last_name,
        'TEAM' => $team_name,
        'YEAR' => date('Y'),
      ],
    ];
    
    try {
      $result = $mailchimp->addOrUpdateListMember($list_id, $email, $member_info);
      return ['success' => TRUE, 'status' => $result['status']];
      
    } catch (\Exception $e) {
      \Drupal::logger('tremblant_mailchimp')->error($e->getMessage());
      return ['success' => FALSE, 'error' => $e->getMessage()];
    }
  }
}
```

#### Batch Subscribe

```php
public function batchSubscribe(array $subscribers) {
  $mailchimp = \Drupal::service('mailchimp.api');
  $list_id = $this->config->get('list_id');
  
  $operations = [];
  foreach ($subscribers as $subscriber) {
    $operations[] = [
      'method' => 'POST',
      'path' => "/lists/{$list_id}/members",
      'body' => json_encode([
        'email_address' => $subscriber['email'],
        'status' => 'subscribed',
        'merge_fields' => $subscriber['fields'],
      ]),
    ];
  }
  
  $batch = [
    'operations' => $operations,
  ];
  
  return $mailchimp->executeBatch($batch);
}
```

### Webhook Integration

**Webhook URL:** `https://24htremblant.com/mailchimp/webhook`

**Events:**
- `subscribe` - User subscribed
- `unsubscribe` - User unsubscribed
- `profile` - Profile updated
- `cleaned` - Email bounced/invalid

```php
public function processWebhook(Request $request) {
  $data = $request->request->all();
  $type = $data['type'];
  $email = $data['data']['email'];
  
  switch ($type) {
    case 'unsubscribe':
      // Update user preferences
      $this->updateUserPreferences($email, ['newsletter' => FALSE]);
      break;
      
    case 'profile':
      // Sync profile changes
      $this->syncProfileUpdate($email, $data['data']);
      break;
  }
}
```

---

## 🔐 Facebook OAuth Authentication

### Overview

Facebook OAuth allows users to register and login using their Facebook account, simplifying the registration process.

**Module:** `social_auth_facebook` (contrib)  
**Integration Type:** OAuth 2.0

### Configuration

#### Facebook App Setup

1. Create app at https://developers.facebook.com
2. Add OAuth redirect URI:
   - `https://24htremblant.com/user/login/facebook/callback`
   - `https://test-24htremblant.pantheonsite.io/user/login/facebook/callback`

#### Drupal Configuration

```yaml
# Configuration: social_auth_facebook.settings

app_id: "1234567890123456"
app_secret: "abcdef1234567890abcdef1234567890"
graph_api_version: "v18.0"
redirect_uri: "https://24htremblant.com/user/login/facebook/callback"
```

### OAuth Flow

```mermaid
sequenceDiagram
    participant User
    participant Drupal
    participant Facebook
    
    User->>Drupal: Click "Login with Facebook"
    Drupal->>Drupal: Generate state token
    Drupal->>Facebook: Redirect to OAuth URL
    Facebook->>User: Show permission dialog
    User->>Facebook: Grant permissions
    Facebook->>Drupal: Redirect with code
    Drupal->>Facebook: Exchange code for token
    Facebook->>Drupal: Return access token
    Drupal->>Facebook: Fetch user profile
    Facebook->>Drupal: Return profile data
    Drupal->>Drupal: Create/link account
    Drupal->>User: Log in user
```

### Implementation

```php
// Drupal handles OAuth automatically via social_auth_facebook module

// Custom event subscriber for post-login
namespace Drupal\tremblant_core\EventSubscriber;

use Drupal\social_auth_facebook\Event\FacebookUserEvent;

class FacebookLoginSubscriber implements EventSubscriberInterface {
  
  public static function getSubscribedEvents() {
    return [
      'social_auth_facebook.user_created' => 'onUserCreated',
      'social_auth_facebook.user_logged_in' => 'onUserLoggedIn',
    ];
  }
  
  public function onUserCreated(FacebookUserEvent $event) {
    $user = $event->getUser();
    $facebook_data = $event->getFacebookData();
    
    // Pre-fill profile with Facebook data
    $profile = \Drupal::entityTypeManager()
      ->getStorage('profile')
      ->create([
        'type' => 'participant',
        'uid' => $user->id(),
        'field_first_name' => $facebook_data['first_name'],
        'field_last_name' => $facebook_data['last_name'],
        'field_facebook_id' => $facebook_data['id'],
      ]);
    $profile->save();
  }
}
```

### Security

```php
// Validate state parameter (CSRF protection)
public function callback(Request $request) {
  $state = $request->get('state');
  $expected_state = $_SESSION['facebook_oauth_state'];
  
  if (!hash_equals($expected_state, $state)) {
    throw new AccessDeniedHttpException('Invalid state parameter');
  }
  
  // Whitelist redirect URIs
  $allowed_redirects = [
    'https://24htremblant.com/user/login/facebook/callback',
    'https://test-24htremblant.pantheonsite.io/user/login/facebook/callback'
  ];
  
  // Continue OAuth flow...
}
```

---

## ☁️ Google Cloud Storage

### Overview

Google Cloud Storage is used for storing large media files and assets.

**Module:** Custom integration  
**Authentication:** Service Account

### Configuration

```yaml
# Service account credentials
service_account:
  project_id: "24h-tremblant"
  private_key: "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
  client_email: "storage@24h-tremblant.iam.gserviceaccount.com"

bucket: "24h-tremblant-media"
```

### Implementation

```php
use Google\Cloud\Storage\StorageClient;

class GoogleCloudStorageService {
  
  protected $storage;
  protected $bucket;
  
  public function __construct() {
    $this->storage = new StorageClient([
      'projectId' => 'your-project-id',
      'keyFilePath' => '/path/to/service-account-key.json'
    ]);
    $this->bucket = $this->storage->bucket('24h-tremblant-media');
  }
  
  public function uploadFile($local_path, $destination) {
    $file = fopen($local_path, 'r');
    $object = $this->bucket->upload($file, [
      'name' => $destination,
      'metadata' => [
        'contentType' => mime_content_type($local_path)
      ]
    ]);
    return $object->info()['mediaLink'];
  }
  
  public function getPublicUrl($filename) {
    return sprintf(
      'https://storage.googleapis.com/%s/%s',
      $this->bucket->name(),
      $filename
    );
  }
}
```

---

## 📊 Google Sheets Reporting

### Overview

Automated export of reports to Google Sheets for analysis and sharing.

**Authentication:** Service Account  
**Frequency:** Nightly via Azure Pipeline cron

### Configuration

```yaml
# Service account with Sheets API access
service_account:
  type: "service_account"
  project_id: "24h-tremblant"
  client_email: "sheets@24h-tremblant.iam.gserviceaccount.com"

spreadsheet_id: "1abc_xyz123_spreadsheet_id"
```

### Export Implementation

```php
use Google\Client;
use Google\Service\Sheets;

class GoogleSheetsExporter {
  
  public function exportDonations($donations) {
    $client = new Client();
    $client->setAuthConfig('/path/to/credentials.json');
    $client->addScope(Sheets::SPREADSHEETS);
    
    $service = new Sheets($client);
    $spreadsheetId = $this->config->get('spreadsheet_id');
    
    // Prepare data
    $values = [
      ['Date', 'Team', 'Amount', 'Donor', 'Email'],
    ];
    
    foreach ($donations as $donation) {
      $values[] = [
        $donation->getCreatedTime()->format('Y-m-d'),
        $donation->getTeam()->label(),
        '$' . number_format($donation->getTotalPrice()->getNumber(), 2),
        $donation->getBillingProfile()->get('field_name')->value,
        $donation->getEmail(),
      ];
    }
    
    // Update sheet
    $body = new Sheets\ValueRange(['values' => $values]);
    $params = ['valueInputOption' => 'RAW'];
    
    $result = $service->spreadsheets_values->update(
      $spreadsheetId,
      'Donations!A1',
      $body,
      $params
    );
    
    return $result->getUpdatedCells();
  }
}
```

### Azure Pipeline Cron

```yaml
# azure-pipelines-reports-cron.yml
schedules:
- cron: "0 3 * * *"  # 3 AM daily
  displayName: Nightly reports export
  branches:
    include:
    - master

steps:
- script: |
    terminus drush 24h-tremblant.live -- tremblant-reports:export-donations
    terminus drush 24h-tremblant.live -- tremblant-reports:export-teams
```

---

## 🤖 reCAPTCHA v3

### Overview

Google reCAPTCHA v3 protects forms from bots without user interaction.

**Module:** `recaptcha_v3` (contrib)  
**Score Threshold:** 0.5

### Configuration

```yaml
# Configuration: recaptcha_v3.settings

site_key: "6LcXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
secret_key: "6LcYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY"
score_threshold: 0.5
```

### Implementation

```php
// Add to forms
$form['recaptcha'] = [
  '#type' => 'recaptcha_v3',
  '#action' => 'donation_form',
];

// Validation
public function validateForm(array &$form, FormStateInterface $form_state) {
  $recaptcha = $form_state->getValue('recaptcha');
  $score = $recaptcha['score'];
  
  if ($score < 0.5) {
    $form_state->setErrorByName('recaptcha', 'Suspicious activity detected.');
  }
}
```

---

## 📈 Google Tag Manager

### Overview

Google Tag Manager manages all analytics and tracking scripts.

**Module:** `google_tag` (contrib)  
**Container ID:** `GTM-XXXXXXX`

### Configuration

```yaml
# Configuration: google_tag.settings

container_id: "GTM-XXXXXXX"
include_environment: false
```

### Custom Events

```javascript
// Track donation
dataLayer.push({
  'event': 'donation',
  'amount': 100.00,
  'team': 'Team A',
  'donor_type': 'anonymous'
});

// Track team registration
dataLayer.push({
  'event': 'team_registration',
  'team_name': 'Les Coureurs',
  'activity': 'Running'
});
```

---

## 🎯 Integration Best Practices

### Error Handling

```php
try {
  $result = $api->call();
} catch (ApiException $e) {
  // Log error
  \Drupal::logger('integration')->error($e->getMessage());
  
  // Graceful degradation
  drupal_set_message('Service temporarily unavailable', 'warning');
  
  // Queue for retry
  $queue = \Drupal::queue('api_retry');
  $queue->createItem(['endpoint' => $endpoint, 'data' => $data]);
}
```

### Rate Limiting

```php
public function makeApiCall() {
  $rate_limiter = \Drupal::service('rate_limiter');
  
  if (!$rate_limiter->check('payfacto_api', 100, 60)) {
    throw new TooManyRequestsException('Rate limit exceeded');
  }
  
  // Make API call
}
```

### Monitoring

```php
// Track API call performance
$start = microtime(TRUE);
$result = $api->call();
$duration = microtime(TRUE) - $start;

if ($duration > 2.0) {
  \Drupal::logger('performance')->warning(
    'Slow API call: @endpoint took @duration seconds',
    ['@endpoint' => $endpoint, '@duration' => $duration]
  );
}
```

---

## 🐛 Troubleshooting

### PayFacto Issues

**Problem:** Payment declined  
**Solution:** Check card details, verify insufficient funds

**Problem:** Webhook not received  
**Solution:** Verify webhook URL, check signature validation

### MailChimp Issues

**Problem:** Subscribers not syncing  
**Solution:** Check API key, verify list ID, run manual sync

### Facebook OAuth Issues

**Problem:** Redirect URI mismatch  
**Solution:** Verify redirect URI in Facebook App settings

---

**Document Version**: 2.0.0  
**Last Updated**: 2026-02-26  
**Maintained By**: GMA-AI-Lab Development Team

---

## 📝 Change Log

| Date | Version | Changes | Author |
|------|---------|---------|--------|
| 2026-02-26 | 2.0.0 | Complete integration documentation with code examples | Dev Team |
| 2026-02-26 | 1.0.0 | Initial integration overview | Dev Team |