# Deploy to Production

Deploy to production environment with safety checks.

## Instructions

1. **Safety checks** (ALL must pass):
   - Verify we're on main/master or a release branch
   - No uncommitted changes: `git status`
   - All tests pass
   - Build succeeds
   - Confirm with user before proceeding
2. Check deployment prerequisites:
   - Required environment variables set
   - Database migrations ready (if applicable)
   - Feature flags configured (if applicable)
3. Detect deployment method (same as staging)
4. **Create a git tag** for the release:
   ```bash
   git tag -a v$(date +%Y%m%d-%H%M) -m "Production release"
   ```
5. Execute deployment
6. Verify deployment health:
   - Check health endpoints
   - Verify critical paths work
7. Report success/failure with rollback instructions

## Production Checklist

Before deploying, verify:
- [ ] All tests passing
- [ ] Code reviewed and approved
- [ ] Database migrations tested
- [ ] Environment variables configured
- [ ] Monitoring/alerts in place
- [ ] Rollback plan ready

## Arguments

$ARGUMENTS - Optional: version tag or deployment flags

## Critical Safety Rules

- **ALWAYS** confirm with user before production deploy
- **NEVER** deploy untested code
- **ALWAYS** create a git tag
- **HAVE** a rollback plan ready

## Examples

- `/deploy:production` - Deploy to production (with confirmation)
- `/deploy:production v2.1.0` - Deploy with specific version tag
