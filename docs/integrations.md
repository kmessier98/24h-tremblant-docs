# API Integrations Guide - 24h Tremblant D9

Comprehensive guide for all third-party API integrations.

## Integration Summary

| Service | Purpose | Status |
|---------|---------|--------|
| **PayFacto** | Payment processing | ✅ Active |
| **MailChimp** | Email marketing | ✅ Active |
| **Facebook OAuth** | Social authentication | ✅ Active |
| **Google Cloud Storage** | File storage | ⚠️ Configured |
| **Google Sheets** | Reporting | ✅ Active |
| **reCAPTCHA v3** | Spam protection | ✅ Active |
| **Google Tag Manager** | Analytics | ✅ Active |

## PayFacto Payment Gateway

### Key Features
- Credit card processing
- Tokenization (PCI-compliant)
- Recurring payments support
- Canadian market focus

### Configuration
- API credentials (merchant ID, API key)
- Payment gateway setup
- Checkout flow integration
- Webhook handling

### Security
- Never store card data
- Validate webhooks
- HTTPS only
- PCI compliance requirements

## MailChimp Email Marketing

### Features
- List management
- Automatic sync
- Campaign management
- Webhook configuration

### API Usage
- Subscribe/unsubscribe
- Update subscriber
- Batch operations

## Facebook OAuth

### Setup
- Facebook App configuration
- Drupal module installation
- OAuth flow
- User profile sync

### Security
- State parameter validation
- Redirect URI whitelist
- Token management

## Google Cloud Storage

### Configuration
- Service account setup
- Credentials management
- Drupal stream wrapper
- File operations

## Google Sheets Reporting

### Features
- Automated report exports
- Azure Pipeline cron jobs
- Drush commands
- Data formatting

## Integration Best Practices

- Error handling
- Rate limiting
- Monitoring
- Graceful degradation

For the complete integrations guide including code examples, API references, and troubleshooting, please see the full document in the repository.

**Document Version**: 1.0  
**Last Updated**: 2026-02-26  
**Maintained By**: GMA-AI-Lab Development Team