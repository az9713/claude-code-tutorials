# Deploy to Staging

Deploy current branch to staging environment.

## Instructions

1. Pre-flight checks:
   - Verify we're on the correct branch: `git branch --show-current`
   - Check for uncommitted changes: `git status`
   - Ensure tests pass: run test suite
   - Ensure build succeeds: run build command
2. Detect deployment method:
   - Check for deployment scripts in `package.json`, `Makefile`, or `scripts/`
   - Look for CI/CD config (`.github/workflows/`, `.gitlab-ci.yml`)
   - Check for platform-specific files (Vercel, Netlify, Railway, etc.)
3. Execute deployment or provide instructions
4. Verify deployment succeeded
5. Report deployment URL/status

## Common Deployment Patterns

| Platform | Typical Command |
|----------|-----------------|
| Vercel | `vercel --env preview` |
| Netlify | `netlify deploy` |
| Railway | `railway up` |
| Heroku | `git push heroku staging:main` |
| AWS | `aws deploy` or CDK/SAM commands |
| Docker | `docker compose -f docker-compose.staging.yml up` |
| Kubernetes | `kubectl apply -f k8s/staging/` |
| Custom | `./scripts/deploy-staging.sh` |

## Arguments

$ARGUMENTS - Optional: specific service or deployment flags

## Safety

- Never deploy uncommitted changes
- Always run tests before deploying
- Confirm before executing destructive operations

## Examples

- `/deploy:staging` - Deploy to staging
- `/deploy:staging --skip-tests` - Deploy without running tests (use carefully)
- `/deploy:staging api` - Deploy only the API service
