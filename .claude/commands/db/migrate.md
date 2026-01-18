# Database Migrations

Run or create database migrations.

## Instructions

1. Detect the ORM/migration tool:
   - `prisma/` directory → Prisma
   - `drizzle.config.*` → Drizzle
   - `migrations/` with `alembic.ini` → Alembic (Python)
   - `db/migrate/` → Rails ActiveRecord
   - `migrations/` with `.sql` files → Raw SQL migrations
   - `sequelize` in package.json → Sequelize
   - `typeorm` in package.json → TypeORM
   - `knex` in package.json → Knex.js
2. Based on arguments:
   - No args: Show migration status
   - `create <name>`: Create new migration
   - `run` or `up`: Run pending migrations
   - `down` or `rollback`: Rollback last migration
   - `status`: Show migration status
   - `reset`: Reset database (with confirmation!)

## Common Migration Commands

| Tool | Status | Run | Rollback | Create |
|------|--------|-----|----------|--------|
| Prisma | `prisma migrate status` | `prisma migrate deploy` | N/A | `prisma migrate dev --name` |
| Drizzle | `drizzle-kit status` | `drizzle-kit push` | N/A | `drizzle-kit generate` |
| Alembic | `alembic current` | `alembic upgrade head` | `alembic downgrade -1` | `alembic revision -m` |
| Knex | `knex migrate:status` | `knex migrate:latest` | `knex migrate:rollback` | `knex migrate:make` |
| TypeORM | `typeorm migration:show` | `typeorm migration:run` | `typeorm migration:revert` | `typeorm migration:create` |

## Arguments

$ARGUMENTS - Subcommand: status, run, up, down, rollback, create <name>, reset

## Safety

- **NEVER** run reset without explicit user confirmation
- **ALWAYS** backup before running migrations in production
- Show what will be migrated before executing

## Examples

- `/db:migrate` - Show migration status
- `/db:migrate run` - Run pending migrations
- `/db:migrate create add_user_roles` - Create new migration
- `/db:migrate rollback` - Rollback last migration
