# Deployment Runbook - 24h Tremblant D9

## 📋 Overview

This runbook provides step-by-step procedures for deploying the 24h Tremblant D9 application to various environments, including rollback procedures and troubleshooting steps.

---

## 🎯 Pre-Deployment Checklist

### For Test Environment Deployment

- [ ] All Azure DevOps work items linked to PR are completed
- [ ] Code review approved by at least one senior developer
- [ ] All CI/CD checks passed (validation pipeline green)
- [ ] Local testing completed and documented
- [ ] Configuration exported (`drush cex` committed)
- [ ] Database updates implemented in `hook_update_N()`
- [ ] No merge conflicts with target branch
- [ ] Changelog updated (if applicable)

### For Production Environment Deployment

- [ ] **All Test Environment checklist items** ✅
- [ ] Test environment validation completed (minimum 24 hours)
- [ ] Client/Stakeholder approval obtained
- [ ] Production database backup verified (< 1 hour old)
- [ ] Deployment scheduled during low-traffic window
- [ ] Rollback plan reviewed and ready
- [ ] On-call personnel notified
- [ ] Status page updated (if applicable)
- [ ] Post-deployment verification plan documented

---

## 🚀 Deployment Procedures

### Test Environment Deployment (Automatic)

**Trigger**: Merge to `master` branch  
**Pipeline**: `azure-pipelines-deploy.yml`  
**Duration**: ~10 minutes

#### Steps (Automated)

The Azure Pipeline automatically:
1. Sets up PHP 8.3 and Node.js 8.x
2. Runs composer install
3. Clones Pantheon repository
4. Rsyncs artifact to Pantheon
5. Runs database updates (drush updb)
6. Imports configuration (drush cim)
7. Clears caches (drush cr)

#### Post-Deployment Verification

```bash
# Check site status
terminus drush 24h-tremblant.test -- status

# Check for errors
terminus drush 24h-tremblant.test -- watchdog:show --severity=Error --count=20

# Verify configuration
terminus drush 24h-tremblant.test -- config:status
```

---

### Production Environment Deployment (Manual)

**Pipeline**: `azure-pipelines-production.yml`  
**Duration**: ~15-20 minutes  
**Recommended Window**: 2:00 AM - 4:00 AM EST (weekdays)

#### Pre-Production Steps

```bash
# Create backup
terminus backup:create 24h-tremblant.live --element=all

# Verify test environment
terminus drush 24h-tremblant.test -- status
terminus drush 24h-tremblant.test -- core:requirements
```

#### Post-Production Verification

Critical tests to execute immediately:

| Test | Expected Result |
|------|-----------------|
| Site accessible | HTTP/2 200 |
| User login | Successful authentication |
| Team creation | Order completes |
| Donation | Payment processes |
| Leaderboard | Rankings display correctly |

---

## ⏪ Rollback Procedures

### When to Rollback

Execute rollback if:
- ❌ Critical errors in watchdog
- ❌ Site inaccessible (500 errors)
- ❌ Payment processing failures
- ❌ Data loss or corruption
- ❌ Performance degradation (> 10s page loads)

### Rollback Option 1: Code Rollback (Fastest - 5 minutes)

```bash
# Get previous commit
terminus env:commits 24h-tremblant.live --limit=5

# Reset and push
git reset --hard <previous-commit-hash>
git push --force pantheon master

# Restore configuration
terminus drush 24h-tremblant.live -- config:import -y
terminus drush 24h-tremblant.live -- cr
```

### Rollback Option 2: Full Restore (15 minutes)

```bash
# List backups
terminus backup:list 24h-tremblant.live

# Restore all elements
terminus backup:restore 24h-tremblant.live --element=code
terminus backup:restore 24h-tremblant.live --element=database
terminus backup:restore 24h-tremblant.live --element=files
```

**⚠️ WARNING**: Database restore loses all data created since backup.

---

## 🐛 Troubleshooting

### Configuration Import Fails

```bash
# Check status
terminus drush 24h-tremblant.live -- config:status

# Partial import
terminus drush 24h-tremblant.live -- config:import --partial -y
```

### PayFacto Payment Failures

```bash
# Check module status
terminus drush 24h-tremblant.live -- pm:list | grep payfacto

# Verify credentials
terminus drush 24h-tremblant.live -- config:get commerce_payment.commerce_payment_gateway.payfacto

# Clear cache
terminus drush 24h-tremblant.live -- cr
```

### Performance Issues

```bash
# Clear all caches
terminus drush 24h-tremblant.live -- cr

# Check Redis
terminus redis:info 24h-tremblant.live

# Enable aggressive caching (temporary)
terminus drush 24h-tremblant.live -- config:set system.performance cache.page.max_age 3600 -y
```

---

## 📊 Deployment Metrics

| Metric | Target |
|--------|--------|
| Deployment Duration | < 15 min |
| Downtime | < 5 min |
| Rollback Rate | < 5% |
| Post-Deployment Errors | 0 critical |
| Page Load Time | < 3 sec |

---

## 📞 Emergency Contacts

| Role | Contact |
|------|---------|
| DevOps Lead | [TBD] |
| Technical Lead | [TBD] |
| Pantheon Support | https://pantheon.io/support |
| PayFacto Support | [TBD] |

---

## 📚 Related Documentation

- [Main README](../README.md)
- [Security Guidelines](./security.md)
- [Testing Strategy](./testing.md)
- [Architecture](./architecture.md)
- [Integrations](./integrations.md)

---

**Version**: 1.0.0  
**Last Updated**: 2026-02-26  
**Maintained By**: DevOps Team