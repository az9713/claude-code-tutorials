# Database Seeding

Seed the database with initial or test data.

## Instructions

1. Detect the seeding mechanism:
   - Check for `prisma/seed.ts` or `prisma/seed.js`
   - Check for `db/seeds/` directory
   - Check for seed script in `package.json`
   - Check for `seeds/` or `fixtures/` directory
   - Check for Django fixtures
2. Based on arguments:
   - No args: Run default seed
   - `dev`: Seed with development data
   - `test`: Seed with test fixtures
   - `<name>`: Run specific seed file

## Common Seed Commands

| Tool | Command |
|------|---------|
| Prisma | `prisma db seed` |
| Rails | `rails db:seed` |
| Django | `python manage.py loaddata` |
| Knex | `knex seed:run` |
| Laravel | `php artisan db:seed` |
| Custom | `npm run seed` or `./scripts/seed.sh` |

## Arguments

$ARGUMENTS - Optional: seed type (dev, test) or specific seed file

## Safety

- **NEVER** run seeds in production without explicit confirmation
- Warn if database already has data
- Offer to truncate/reset first if needed

## Examples

- `/db:seed` - Run default seed
- `/db:seed dev` - Seed with development data
- `/db:seed --fresh` - Reset and reseed
- `/db:seed users` - Run only user seeds
