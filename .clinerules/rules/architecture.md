# 🏗️ ARCHITECTURE RULES

---

## FRONTEND ARCHITECTURE

### Layering

```
presentation/ → UI components, pages
domain/       → Business logic, entities, use cases
data/         → API calls, repositories, models
shared/       → Common utilities, shared types
```

| Rule | Description |
|------|-------------|
| Dependency direction | Inner layers never import from outer layers |
| Domain isolation | `domain/` has zero framework imports |
| Data isolation | `data/` only imports from `domain/` |
| Presentation | May import from `domain/`, `data/`, `shared/` |

### Component Design

| Rule | Description |
|------|-------------|
| Single responsibility | One component = one job |
| Container / Presenter | Split stateful logic from pure rendering |
| Size limits | .ts ≤ 200 lines, .html ≤ 100 lines → split if exceeded |
| Composition > inheritance | Compose behavior; avoid `extends` |
| Dependency injection | Depend on abstractions (interfaces/tokens), not concrete classes |

### State Management

| Rule | Description |
|------|-------------|
| Single source of truth | One owner per piece of state |
| Unidirectional flow | State down, events up |
| Immutable updates | Never mutate state directly |
| Derived state | Compute from source, don't duplicate |

### File Organization

```
feature-name/
├── components/           # UI components
│   └── feature-card/
│       ├── feature-card.ts
│       ├── feature-card.html
│       └── feature-card.spec.ts
├── services/             # Business logic
│   └── feature.service.ts
├── models/               # Types & interfaces
│   └── feature.model.ts
├── index.ts              # Public API
└── README.md             # Feature docs
```

---

## BACKEND ARCHITECTURE

### Layering

```
routes/       → HTTP handlers, request/response mapping
services/     → Business logic, use cases
repositories/ → Data access, database queries
models/       → Entities, DTOs, schemas
middleware/   → Auth, logging, validation, rate limiting
shared/       → Common utilities, types
```

| Rule | Description |
|------|-------------|
| Routes are thin | Routes only parse requests, call services, return responses |
| Business logic in services | All domain rules live in the service layer |
| Repositories are the data boundary | No raw SQL or ORM calls outside repositories |
| Downward dependency | `routes → services → repositories` only |

### API Design

| Rule | Description |
|------|-------------|
| RESTful conventions | Use nouns for resources, HTTP methods for actions |
| Consistent response format | All endpoints return the same envelope shape |
| Versioning | `/api/v1/...` prefix |
| Pagination | Return `total`, `page`, `pageSize` for list endpoints |
| Error responses | Standard error format with `code`, `message`, `details` |

### Database

| Rule | Description |
|------|-------------|
| Migrations checked in | All schema changes are versioned |
| Never auto-sync | No automatic schema sync in production |
| Indexes are intentional | Add indexes based on query patterns, not guesses |
| Backup strategy | Automated backups with verified restore process |

---

## GENERAL PRINCIPLES

### SOLID

| Principle | Application |
|-----------|-------------|
| **S**ingle Responsibility | 1 module / 1 class = 1 reason to change |
| **O**pen/Closed | Open for extension, closed for modification |
| **L**iskov Substitution | Subtypes must be substitutable for their base types |
| **I**nterface Segregation | Small, focused interfaces > large, general ones |
| **D**ependency Inversion | Depend on abstractions, not concretions |

### Module / Package Design

| Rule | Description |
|------|-------------|
| High cohesion | Things that change together stay together |
| Low coupling | Minimize inter-module dependencies |
| Public API | Each module exposes only what's needed via barrel exports |
| Circular deps | Forbidden — detected at build time |
| Feature-based | Group by feature, not by type |

### Error Handling

| Rule | Description |
|------|-------------|
| Fail early | Validate inputs at boundaries |
| User-friendly | Catch errors at UI layer, show meaningful messages |
| Log thoroughly | Log errors with context at the data/service layer |
| Retry & degrade | Handle transient failures gracefully |
| Never swallow | At minimum, log every caught error |

### Security

| Rule | Description |
|------|-------------|
| Validate all input | Never trust client-side validation alone |
| Least privilege | Services and users get minimum necessary permissions |
| Secrets in config | No secrets in source code; use environment variables or vault |
| HTTPS everywhere | Enforce HTTPS in production |
| CORS is explicit | Whitelist origins, never use wildcard in production |