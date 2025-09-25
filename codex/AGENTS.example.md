
# Project: E-Commerce API

## Tech Stack
- Node.js 20 + TypeScript 5.3 (strict mode)
- NestJS 10 + PostgreSQL 15
- Docker + pnpm (not npm!)

## Quick Start
```bash
pnpm install && docker-compose up -d
pnpm run start:dev  # http://localhost:3000
```

## Project Structure
- `src/modules/*` - Business modules (auth, payment, order)
- `src/core/*` - Framework code (don't modify)
- `src/shared/*` - Shared DTOs and utils

## Key Commands
- `pnpm test:watch` - Test during development
- `pnpm run migration:create` - New migration
- `pnpm run build` - Production build

## Critical Rules
1. NEVER commit .env files
2. All money calculations use Decimal type
3. Use transactions for multi-table updates
4. New endpoints need @ApiDoc decorator
5. Test coverage must be >80%

## Code Style
- Controllers: thin, delegates to services
- Services: business logic only
- Repositories: database queries only
- File naming: kebab-case.type.ts
- Commit format: type(scope): message

## Before You Code
- Check existing patterns in `src/modules/auth/*`
- Run tests first to ensure clean state
- Use dependency injection, not imports

## Common Issues
- "Cannot find module" → run `pnpm install`
- Migration fails → check Docker postgres is running
- Timeout errors → increase Jest timeout in test files

## Get Help
- Architecture decisions: `docs/adr/`
- API docs: http://localhost:3000/api
- Team conventions: `docs/team-guide.md`
