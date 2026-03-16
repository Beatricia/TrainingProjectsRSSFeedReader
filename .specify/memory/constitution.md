<!--
SYNC IMPACT REPORT
==================
Version Change: N/A → 1.0.0 (Initial ratification)

Modified Principles: None (initial creation)

Added Sections:
- Core Principles (5 principles)
- Technology Standards
- Development Workflow
- Governance

Removed Sections: None

Templates Status:
- .specify/templates/plan-template.md: ✅ Compatible (uses Constitution Check gate)
- .specify/templates/spec-template.md: ✅ Compatible (requirements align with principles)
- .specify/templates/tasks-template.md: ✅ Compatible (phase structure supports principles)

Follow-up TODOs: None
-->

# RSS Feed Reader Constitution

## Core Principles

### I. Separation of Concerns (API-UI Boundary)

The backend (ASP.NET Core Web API) and frontend (Blazor WebAssembly) MUST remain strictly separated:

- **Backend responsibility**: Data management, feed parsing, business logic, API endpoints
- **Frontend responsibility**: UI rendering, user interaction, API consumption
- All data exchange between frontend and backend MUST occur via HTTP API calls
- Blazor components MUST NOT contain business logic beyond UI state management
- Shared DTOs/models for API contracts MUST be defined in a common location

**Rationale**: Clean separation enables independent testing, future technology swaps, and clear ownership boundaries.

### II. Security-by-Design

All code MUST treat external input as untrusted and implement defensive measures:

- **Input validation**: All user inputs (feed URLs, user actions) MUST be validated before processing
- **URL handling**: Feed URLs MUST be validated for proper format and safe protocols (http/https only)
- **Content safety**: Parsed feed content MUST be sanitized before display to prevent XSS attacks
- **Error exposure**: Error messages MUST NOT leak internal implementation details or stack traces to users
- **Dependency security**: Third-party packages (e.g., System.ServiceModel.Syndication) MUST be from trusted sources and kept updated

**Rationale**: Even a proof-of-concept establishes security patterns that persist as the project grows.

### III. Testable Architecture

Code MUST be structured for testability through dependency injection and interface abstractions:

- All services MUST be registered via ASP.NET Core's built-in DI container
- External dependencies (feed fetching, storage) MUST be accessed through interfaces
- Storage MUST be abstracted behind `ISubscriptionRepository` (or similar) to enable swapping in-memory for persistence later
- HTTP clients for feed fetching MUST use `IHttpClientFactory` pattern
- Unit tests MUST be possible without network access or external resources

**Rationale**: Testable code is maintainable code; interface abstractions enable MVP-to-production evolution without rewrites.

### IV. Code Quality Standards

All code MUST follow consistent quality standards for readability and maintainability:

- **Naming**: Use C# naming conventions (PascalCase for public members, camelCase for private fields with underscore prefix)
- **Documentation**: Public APIs MUST have XML documentation comments describing purpose and parameters
- **Error handling**: Use structured exception handling; catch specific exceptions, not generic `Exception`
- **Null safety**: Use nullable reference types (`#nullable enable`) and handle nullability explicitly
- **Code organization**: One class per file, logical folder structure matching namespaces
- **No dead code**: Unused code, unreachable branches, and commented-out code MUST be removed

**Rationale**: Consistent code reduces cognitive load and prevents maintenance debt accumulation.

### V. Progressive Enhancement (MVP-First)

Implementation MUST follow MVP-first principles with architectural hooks for future growth:

- Start with the simplest working implementation for each feature
- In-memory storage is acceptable for MVP but MUST use repository pattern for easy persistence upgrade
- Defer features explicitly marked as "Post-MVP" (background polling, persistence, etc.)
- YAGNI: Do not implement features "just in case" they might be needed
- Architecture decisions MUST support stated Extended-MVP and Post-MVP goals without requiring rewrites
- Template cleanup: Remove unused Blazor template components before first commit

**Rationale**: Shipping working software quickly while maintaining a clean path for evolution.

## Technology Standards

### Stack Requirements

| Layer | Technology | Version Requirement |
|-------|------------|---------------------|
| Backend | ASP.NET Core Web API | .NET 6+ (LTS preferred) |
| Frontend | Blazor WebAssembly | Matches backend .NET version |
| Feed Parsing | System.ServiceModel.Syndication | Extended-MVP phase |
| Storage (MVP) | In-memory List<T> | Thread-safe if concurrent access possible |
| Storage (Post-MVP) | TBD (SQLite, EF Core) | Repository pattern enables swap |

### Project Structure

```
/
├── src/
│   ├── RSSReader.Api/           # ASP.NET Core Web API project
│   │   ├── Controllers/         # API endpoints
│   │   ├── Services/            # Business logic
│   │   └── Models/              # Domain models
│   ├── RSSReader.Web/           # Blazor WebAssembly project
│   │   ├── Pages/               # Razor pages/routes
│   │   ├── Components/          # Reusable UI components
│   │   └── Services/            # API client services
│   └── RSSReader.Shared/        # Shared DTOs and contracts
├── tests/
│   ├── RSSReader.Api.Tests/     # Backend unit/integration tests
│   └── RSSReader.Web.Tests/     # Frontend component tests
└── docs/                        # Additional documentation
```

### Cross-Platform Compatibility

- All code MUST build and run on Windows, macOS, and Linux
- Use `Path.Combine()` for file paths, not hardcoded separators
- Avoid platform-specific APIs unless abstracted behind interfaces

## Development Workflow

### Quality Gates

Before code is considered complete:

1. **Builds without warnings**: `dotnet build` produces no warnings (treat warnings as errors in CI)
2. **Tests pass**: All unit tests pass with `dotnet test`
3. **Follows principles**: Code review verifies compliance with constitution principles
4. **No TODOs without issues**: Any `TODO` comments MUST reference a tracked issue/task

### Code Review Checklist

Reviewers MUST verify:

- [ ] Separation of concerns maintained (API doesn't render UI, UI doesn't contain business logic)
- [ ] External inputs validated and sanitized
- [ ] Interfaces used for testability (not concrete implementations for external dependencies)
- [ ] Naming conventions followed
- [ ] No dead code or unused imports
- [ ] Public APIs documented

### Error Handling Pattern

```csharp
// Required pattern for service methods
public async Task<Result<T>> OperationAsync()
{
    try
    {
        // Operation logic
        return Result<T>.Success(value);
    }
    catch (SpecificException ex)
    {
        _logger.LogError(ex, "Context-specific message");
        return Result<T>.Failure("User-friendly error message");
    }
}
```

## Governance

### Constitution Authority

This constitution supersedes all other development practices for the RSS Feed Reader project. All implementation decisions MUST align with these principles.

### Amendment Process

1. Proposed amendments MUST be documented with rationale
2. Breaking changes to principles require justification and migration plan
3. Version increments follow semantic versioning:
   - **MAJOR**: Principle removal or incompatible redefinition
   - **MINOR**: New principle or material guidance expansion
   - **PATCH**: Clarifications, wording improvements

### Compliance Verification

- All pull requests MUST include a constitution compliance statement
- Code reviews MUST verify principle adherence
- Complexity that appears to violate principles MUST be documented with explicit justification

**Version**: 1.0.0 | **Ratified**: 2026-03-16 | **Last Amended**: 2026-03-16
