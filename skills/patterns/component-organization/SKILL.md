---
name: component-organization
description: Frontend directory structure and file organization patterns. Apply when creating new components or restructuring frontend code.
type: pattern
---

# Frontend Component Organization

**This is a reference pattern.** Learn from the approach, adapt to your context — don't copy verbatim.

**Problem**: Frontend projects grow into flat directories with hundreds of files, making it hard to find related code or understand feature boundaries.

**Solution**: Organize by feature/domain, colocate related files, and separate shared utilities from feature-specific code.

---

## Pattern

**Principle**: Organize by feature or domain, not by file type.

```
frontend/
├── app/                 # Framework routing (Next.js app dir, Nuxt pages, etc.)
│   ├── layout.tsx
│   └── page.tsx
├── components/          # Reusable UI components
│   ├── ui/              # Base primitives (Button, Input, Modal, etc.)
│   ├── header.tsx
│   ├── footer.tsx
│   └── ...
├── lib/                 # Feature modules and hooks
│   ├── auth/            # Authentication context, hooks, utilities
│   ├── notifications/   # Notification service, hooks
│   └── dashboard/       # Dashboard-specific logic
├── queries/             # Data fetching (GraphQL queries, API calls)
├── shared/              # Cross-cutting types, constants, helpers
│   ├── types.ts
│   └── constants.ts
└── public/              # Static assets
```

**Key Rules**:
- **Feature directories** in `lib/` contain hooks, services, and utilities for one domain
- **Components** that belong to a single feature live in that feature's `lib/` directory
- **Shared components** used across features live in `components/`
- **Base UI primitives** (buttons, inputs, modals) live in `components/ui/`

---

## Why This Pattern?

**Benefits**:
- **Discoverability**: Related code lives together — find a feature's hook, service, and types in one directory
- **Encapsulation**: Feature boundaries are visible in the file tree
- **Scalability**: Adding a feature means adding a directory, not scattering files across the tree
- **Deletability**: Removing a feature is removing a directory

**Anti-pattern — organizing by file type**:
```
# ❌ Hard to find related code
components/
  DashboardChart.tsx
  NotificationBell.tsx
  UserProfile.tsx
hooks/
  useDashboard.ts
  useNotifications.ts
  useProfile.ts
services/
  dashboardService.ts
  notificationService.ts
  profileService.ts
```

---

## Feature Module Structure

Each feature directory in `lib/` follows a consistent internal structure:

```
lib/auth/
├── auth-context.tsx      # React context provider
├── use-auth.ts           # Primary hook for consumers
├── auth-utils.ts         # Pure helper functions
└── types.ts              # Feature-specific types (optional)
```

**Conventions**:
- One primary hook per feature (`use-{feature}.ts`) — this is the public API
- Context providers live alongside their hooks
- Utility functions are pure and testable
- Feature-specific types stay in the feature directory; shared types go to `shared/types.ts`

---

## Data Fetching Layer

Separate data fetching definitions from feature logic:

```
queries/
├── users.ts              # User-related queries/mutations
└── dashboard.ts          # Dashboard data queries
```

Features consume these through hooks:

```typescript
// lib/dashboard/use-dashboard.ts
import { getDashboardData } from '@/queries/dashboard';

export function useDashboard() {
  const { data, loading, error } = useQuery(getDashboardData);
  return { metrics: data?.metrics, isLoading: loading };
}
```

This keeps data fetching definitions reusable across features while keeping consumption logic colocated with the feature.

---

## When to Split

- **New directory in `lib/`**: When a feature has 2+ files (hook + service, or hook + context)
- **New file in `components/`**: When a component is used by 2+ features
- **Move to `components/ui/`**: When a component is purely presentational with no business logic

---

## Related Patterns

- Core Principles — Separation of concerns, pure functions
- Frontend Code Quality — Code style within components
