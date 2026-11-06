
# Anvil — AGENTS.md

## 1. Project Overview

Anvil is a reusable TypeScript backend infrastructure toolkit, initially focused on NestJS.

Its purpose is to provide production-ready, composable building blocks for backend applications without imposing a specific application architecture or business domain.

Anvil is intended to extract and consolidate proven patterns from applications such as Seshat and Gilde.

### Vision

Provide developers with reliable, reusable infrastructure so they can focus on building applications rather than repeatedly implementing common backend capabilities.

### Core Principles

- Modular: Every capability should be independently usable.
- Composable: Applications choose which modules they need.
- Framework-aware: First-class NestJS integration.
- Extensible: Allow custom implementations and providers.
- Framework-independent where practical: Keep business logic and contracts separate from NestJS.
- Type-safe: Leverage TypeScript's type system.
- Production-ready: Security, testing, observability and maintainability are first-class concerns.
- Minimal: Avoid unnecessary abstractions and dependencies.
- Open: Prefer standard protocols and interoperable implementations.

Anvil provides infrastructure, not application architecture.

## 2. Architecture

Anvil consists of independently versioned packages.

Proposed structure:

```text
anvil/
├── packages/
│   ├── core/
│   ├── auth/
│   ├── organization/
│   ├── rbac/
│   ├── database/
│   ├── events/
│   └── ai/
├── examples/
├── docs/
├── package.json
├── pnpm-workspace.yaml
└── AGENTS.md
```

Package names and npm scope should be configurable until publishing conventions are finalized.

## 3. Package Responsibilities

### Core

Shared infrastructure used across Anvil packages.

Responsibilities:
- Common interfaces and types.
- Configuration utilities.
- Request context.
- Correlation IDs.
- Logging abstractions.
- Exception handling.
- Health check abstractions.
- Shared decorators and utilities where appropriate.

Core must remain lightweight and avoid dependencies on higher-level packages.

### Auth

Authentication mechanisms and identity integration.

Responsibilities:
- JWT validation.
- JWKS integration.
- OIDC compatibility.
- Authentication guards.
- Current user decorators.
- Public route decorators.
- Authentication context.
- Pluggable authentication strategies.

Authentication should be based on standard protocols.

Do not tightly couple Anvil to a particular identity provider.

Authentication answers:

**Who is making this request?**

It does not determine what the user is allowed to do.

### Organization

Multi-tenancy and organization context.

Responsibilities:
- Organization resolution.
- Organization context.
- Organization guards.
- Organization decorators.
- Membership contracts.
- Tenant isolation utilities.
- Organization-aware request context.

Organization resolution must be configurable.

Applications may use:
- HTTP headers.
- JWT claims.
- Subdomains.
- Custom resolvers.

Anvil must not impose a specific organization database schema.

Organization context answers:

**Which organization is the request operating in?**

### RBAC

Authorization and access control.

Responsibilities:
- Roles.
- Permissions.
- Permission decorators.
- Authorization guards.
- Policy evaluation.
- Resource-level authorization.
- Extensible authorization strategies.

Use permissions as the fundamental authorization primitive.

Roles should be collections of permissions rather than hardcoded authorization rules.

Support future ABAC and policy-based authorization without requiring it initially.

Authorization answers:

**What is the user allowed to do?**

### Database

Reusable database infrastructure.

Responsibilities:
- Pagination.
- Filtering.
- Sorting.
- Transaction abstractions where justified.
- Database integration utilities.
- Repository-related abstractions where appropriate.

Do not force a specific ORM or database technology.

Avoid introducing generic repository abstractions without a demonstrated use case.

### Events

Application event infrastructure.

Responsibilities:
- Event contracts.
- Event metadata.
- Event handlers.
- Event dispatching abstractions.
- Correlation and causation identifiers.
- Messaging integration.
- Outbox pattern support where appropriate.

Distinguish between in-process application events and distributed integration events.

Do not assume that every application needs distributed messaging.

### AI

AI-related backend infrastructure.

Responsibilities:
- Provider abstractions.
- Structured output utilities.
- Streaming abstractions.
- Tool integration.
- AI configuration.
- Common AI-related contracts.

Avoid coupling the package to a single AI provider or framework.

Integrations should remain optional.

## 4. Dependency Rules

Follow these dependency principles:

1. Core must not depend on other Anvil packages.
2. Higher-level packages may depend on Core.
3. Packages should not depend on unrelated packages.
4. Circular dependencies are prohibited.
5. NestJS-specific implementations should be separated from framework-independent contracts where practical.
6. External integrations must be optional whenever possible.

Prefer dependency inversion over direct implementation dependencies.

Applications should be able to replace implementations without modifying Anvil internals.

## 5. NestJS Development Guidelines

Anvil should follow established NestJS conventions.

### Modules

- Expose functionality through NestJS modules.
- Support dynamic modules when configuration is required.
- Prefer `forRoot()` and `forRootAsync()` for configurable modules.
- Export only necessary providers.
- Avoid globally registered modules unless explicitly justified.
- Support dependency injection and custom providers.

Example:

```typescript
@Module({})
export class AnvilAuthModule {
  static forRoot(
    options: AnvilAuthOptions,
  ): DynamicModule {
    return {
      module: AnvilAuthModule,
      providers: [
        {
          provide: ANVIL_AUTH_OPTIONS,
          useValue: options,
        },
      ],
      exports: [
        ANVIL_AUTH_OPTIONS,
      ],
    };
  }
}
```

### Configuration

- Validate configuration at startup.
- Provide sensible defaults where appropriate.
- Fail fast on invalid configuration.
- Support asynchronous configuration.
- Never expose secrets through logs.

### Decorators and Guards

- Use metadata decorators for route configuration.
- Keep guards focused on a single responsibility.
- Make authorization behavior explicit.
- Avoid hidden global behavior.

## 6. Authentication and Security

Security is a fundamental requirement.

- Validate JWT signatures.
- Validate issuer and audience.
- Validate expiration and other relevant claims.
- Support JWKS key rotation.
- Never trust unverified JWT claims.
- Never accept organization context without validating membership or authorization.
- Avoid storing authentication state in global mutable variables.
- Never log access tokens, refresh tokens or secrets.
- Use secure defaults.
- Fail closed when authorization cannot be established.

Authentication and authorization must remain separate concerns.

## 7. Multi-tenancy

Tenant isolation must be enforced consistently.

- Every organization-scoped operation must have an explicit organization context.
- Never rely exclusively on frontend organization selection.
- Never trust an organization ID supplied by a client without validation.
- Validate user membership before granting access.
- Ensure database queries are scoped appropriately.
- Prevent cross-organization data access.
- Make tenant context available through dependency injection.

Do not assume all applications use the same tenancy model.

Support customization without weakening isolation.

## 8. RBAC Design

Prefer granular permissions.

Example:

```typescript
const permissions = [
  'projects:read',
  'projects:write',
  'projects:delete',
  'members:read',
  'members:manage',
];
```

Example usage:

```typescript
@RequirePermissions('projects:read')
@Get()
findAll() {
  return this.projectsService.findAll();
}
```

Multiple permissions should have explicitly documented evaluation semantics.

Distinguish between:
- Authentication.
- Organization membership.
- Role assignment.
- Permission evaluation.
- Resource-level authorization.

Do not assume that a role name has universal meaning across applications.

## 9. TypeScript Guidelines

- Enable strict TypeScript settings.
- Avoid `any`.
- Prefer explicit interfaces and types for public APIs.
- Use generics where they improve type safety.
- Avoid unnecessary type assertions.
- Prefer composition over inheritance.
- Keep public APIs small and predictable.
- Use consistent naming conventions.
- Avoid exposing internal implementation details.

Prefer immutable data structures where practical.

Public package APIs should be intentionally designed and documented.

## 10. Testing

All packages must have automated tests.

### Unit Tests

Test:
- Core business logic.
- Guards and decorators.
- Configuration validation.
- Error handling.
- Permission evaluation.
- Organization resolution.

### Integration Tests

Test:
- NestJS module registration.
- Dependency injection.
- Authentication flows.
- Organization context propagation.
- Authorization behavior.
- Database integrations where applicable.

### Security Tests

Explicitly test:
- Invalid tokens.
- Expired tokens.
- Invalid issuers and audiences.
- Missing organization membership.
- Cross-organization access.
- Missing permissions.
- Unauthorized resource access.

Tests should verify both successful and unsuccessful scenarios.

## 11. Documentation

Every package must document:

- Purpose.
- Installation.
- Configuration.
- Usage examples.
- Public APIs.
- Extension points.
- Dependencies.
- Security considerations.

Provide practical examples that can be copied into real NestJS applications.

Keep documentation aligned with actual implementation.

## 12. Versioning and Releases

Each package should be independently versioned.

Use semantic versioning:

- MAJOR: Breaking changes.
- MINOR: Backward-compatible functionality.
- PATCH: Backward-compatible fixes.

Recommended release practices:

- Automated builds.
- Automated tests.
- Package-level changelogs.
- Changesets or equivalent release tooling.
- Explicit public API compatibility.
- Automated npm publishing.

Avoid unnecessary breaking changes.

## 13. Development Workflow

When implementing a feature:

1. Identify the actual use case.
2. Check whether Seshat or Gilde already has a working implementation.
3. Identify reusable contracts and behavior.
4. Define the smallest useful public API.
5. Implement the functionality.
6. Add unit and integration tests.
7. Document the feature.
8. Validate package boundaries.
9. Verify compatibility with consuming applications.

Prefer extracting proven functionality over designing speculative abstractions.

## 14. Avoid Overengineering

Anvil should evolve through real usage.

Do not:

- Build abstractions without a concrete use case.
- Introduce unnecessary dependencies.
- Create a package for every small utility.
- Force applications into a specific architecture.
- Implement features simply because they are theoretically useful.
- Duplicate functionality already provided well by NestJS.
- Build a custom framework where a small integration would suffice.

When in doubt, prefer the simplest solution that solves the actual problem.

## 15. Compatibility and Adoption

Anvil should be usable in multiple independent applications.

Initial validation targets:

- Seshat.
- Gilde.

New modules should be evaluated against actual integration requirements.

An application should be able to adopt one Anvil package without adopting the entire ecosystem.

Avoid assumptions about:
- Database technology.
- Deployment infrastructure.
- Authentication provider.
- Application architecture.
- Messaging infrastructure.
- AI provider.

## 16. Definition of Done

A feature is considered complete when:

- Its public API is clearly defined.
- Its implementation follows package boundaries.
- TypeScript compilation succeeds.
- Relevant tests pass.
- Security implications have been considered.
- Documentation exists.
- Dependencies are justified.
- The package can be consumed independently.
- At least one practical usage example exists.

---

## Guiding Principle

**Anvil provides the tools. Applications retain the freedom to decide how to use them.**
