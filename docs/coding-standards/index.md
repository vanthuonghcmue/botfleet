# 📚 Botfleet Coding Standards Index

> **Created**: November 2025
> **Purpose**: Ensure consistency and quality across the monorepo

## 🎯 Quick Reference

This directory contains comprehensive coding standards for the Botfleet monorepo. These standards are enforced through ESLint, Prettier, and code reviews.

## 📖 Documentation Structure

### Core Standards
- **[README.md](./README.md)** - Monorepo-wide coding standards
  - General principles, naming conventions, TypeScript rules
  - Import organization, error handling, logging
  - Testing, documentation, and Git conventions

### Package-Specific Standards
- **[api-standards.md](./api-standards.md)** - API package guidelines
  - Fastify patterns, controller/service/repository layers
  - Dependency injection with Awilix
  - SSE implementation, database standards

- **[frontend-standards.md](./frontend-standards.md)** - Frontend package guidelines
  - Next.js 15 App Router patterns
  - React components, hooks, and state management
  - TailwindCSS styling, SSE client implementation

- **[worker-standards.md](./worker-standards.md)** - Worker package guidelines
  - BullMQ job processing patterns
  - Multi-location support, retry strategies
  - Performance monitoring, observability

- **[shared-standards.md](./shared-standards.md)** - Shared package guidelines
  - Type definitions, DTOs, validation schemas
  - Bundle size management (must stay lightweight!)
  - Export organization for tree-shaking

## 🚀 Getting Started

### For New Developers

1. **Read the general standards** in [README.md](./README.md)
2. **Focus on your package's standards** based on what you're working on
3. **Run linting** before committing: `pnpm lint`
4. **Use auto-fix** when available: `pnpm lint:fix`

### Quick Commands

```bash
# Lint all packages
pnpm lint

# Auto-fix lint issues
pnpm lint:fix

# Format with Prettier
pnpm format

# Type checking
pnpm type-check

# Run all checks
pnpm check-all
```

## ✅ Key Rules Summary

### Universal Rules
- ✅ **TypeScript strict mode** - No `any` types
- ✅ **kebab-case** for file names
- ✅ **PascalCase** for classes and types
- ✅ **camelCase** for variables and functions
- ✅ **No console.log** in production code
- ✅ **500 lines max** per file

### API Package
- ✅ **No try-catch in controllers** - Let Fastify handle errors
- ✅ **Thin controllers** - Business logic in services
- ✅ **Database ownership** - All migrations happen here

### Frontend Package
- ✅ **Server Components by default** - Use 'use client' only when needed
- ✅ **React Query for server state** - Zustand for client state
- ✅ **TailwindCSS only** - No inline styles or CSS-in-JS

### Worker Package
- ✅ **Stateless processors** - Any worker can handle any job
- ✅ **Let BullMQ handle retries** - Don't catch and suppress errors
- ✅ **Location-based queues** - Support multi-region processing

### Shared Package
- ✅ **Minimal dependencies** - Only Zod allowed
- ✅ **< 50KB bundle size** - Keep it lightweight for frontend
- ✅ **Tree-shakeable exports** - Support optimal bundling

## 🔧 Enforcement

These standards are enforced through:

1. **ESLint** (`.eslintrc.js`) - Automated linting rules
2. **Prettier** (`.prettierrc`) - Consistent formatting
3. **TypeScript** (`tsconfig.json`) - Type checking
4. **Husky** (pre-commit hooks) - Automated checks
5. **CI/CD** - Blocks PRs that violate standards

## 📊 Architecture Decisions

### Why These Standards?

1. **Consistency** - Same patterns everywhere reduces cognitive load
2. **Maintainability** - Clear structure makes changes easier
3. **Performance** - Standards prevent common performance pitfalls
4. **Quality** - Automated enforcement catches issues early
5. **Onboarding** - Clear guidelines help new developers

### Trade-offs Accepted

- **Stricter rules** over flexibility - Consistency wins
- **Explicit over implicit** - More verbose but clearer
- **Duplication in Worker** - Services copied from API for independence
- **Bundle size limits** - Shared package must stay small

## 🔄 Updates and Changes

Standards evolve with the codebase. To propose changes:

1. Create a PR with the proposed change
2. Update both the standard document and enforcement rules
3. Get team consensus
4. Update all existing code to match (if feasible)

## 📝 Checklist for Code Reviews

Before approving a PR, verify:

- [ ] Follows naming conventions
- [ ] No `any` types (except tests)
- [ ] Appropriate error handling
- [ ] Includes tests for new features
- [ ] Documentation updated if needed
- [ ] No console.log statements
- [ ] Passes all lint checks
- [ ] Bundle size impact (for shared package)

## 🚨 Common Violations to Watch For

1. **Using `any` type** - Always define proper types
2. **Try-catch in Fastify controllers** - Let framework handle
3. **Business logic in controllers** - Move to services
4. **Heavy dependencies in shared** - Keep it lightweight
5. **Missing JSDoc on public APIs** - Document public interfaces
6. **Inline styles in React** - Use TailwindCSS classes
7. **Console.log in production** - Use proper logging
8. **Direct database access in controllers** - Use repositories

## 💡 Best Practices

### When in Doubt

1. **Look at existing code** - Follow established patterns
2. **Check the standards** - Documents have examples
3. **Ask the team** - Discuss in PR reviews
4. **Prefer simple** - Clever code is hard to maintain
5. **Think about testing** - If it's hard to test, redesign

### Performance Considerations

- **API**: Use database indexes, batch operations
- **Frontend**: Server Components, lazy loading, memoization
- **Worker**: Parallel processing, appropriate concurrency
- **Shared**: Minimize bundle size, tree-shakeable exports

## 📚 Resources

### External Documentation
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Next.js Documentation](https://nextjs.org/docs)
- [Fastify Documentation](https://www.fastify.io/)
- [BullMQ Documentation](https://docs.bullmq.io/)

### Internal Documentation
- [Architecture Overview](../architecture.md)
- [Tech Stack](../architecture/tech-stack.md)

---

**Remember**: These standards are here to help, not hinder. If something doesn't make sense for a specific case, discuss it with the team!