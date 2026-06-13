---

name: frontend-init-project
description: >
Feature-first React architecture using role folders, HLC (High-Level Component)
composition, concrete imports, and scalable module boundaries.
Use when creating a new frontend project, feature, page, component, hook,
store, API integration, or project structure.
---------------------------------------------

# Frontend Project Architecture

## Golden Rules

1. **Feature-first architecture** — organize by business capability, not technical type.
2. **Downward-only dependencies** — `app → features → shared → infra`.
3. **Single router ownership** — all routes registered in `app/router.tsx`.
4. **Concrete imports only** — never import through feature barrels.
5. **No cross-feature dependencies** — shared logic belongs in `shared/`.
6. **Server state ≠ Client state**

   * Server state → TanStack Query
   * Client state → Zustand
7. **Page components compose only**

   * No API calls
   * No business logic
   * No state orchestration

---

# Project Structure

```text
src/
├── app/
│   ├── router.tsx
│   ├── providers/
│   └── layouts/
│
├── features/
│   ├── auth/
│   ├── dashboard/
│   └── ...
│
├── shared/
│   ├── ui/
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   ├── types/
│   └── constants/
│
└── infra/
    ├── api/
    ├── config/
    ├── i18n/
    └── services/
```

---

# Feature Structure

```text
features/<feature-name>/
│
├── FeaturePage.tsx
│
├── components/
│   ├── FeatureCard.tsx
│   │
│   └── FeatureSection/
│       ├── index.tsx
│       └── components/
│           ├── Header.tsx
│           ├── Content.tsx
│           └── Footer.tsx
│
├── hooks/
│   └── useFeature.ts
│
├── stores/
│   └── feature.store.ts
│
├── types/
│   └── feature.types.ts
│
├── schemas/
│   └── feature.schema.ts
│
├── services/
│   └── feature.service.ts
│
├── __tests__/
│
└── __componentTests__/
```

---

# HLC (High-Level Component)

Use HLC when a component becomes a small feature by itself.

## Simple Component

```text
components/
└── UserCard.tsx
```

## HLC Component

```text
components/
└── UserProfile/
    ├── index.tsx
    └── components/
        ├── Avatar.tsx
        ├── UserInfo.tsx
        └── UserActions.tsx
```

### HLC Rules

* Public entry = `index.tsx`
* Internal parts stay inside `components/`
* Internal components are private
* HLC can contain hooks/types if complexity grows
* Prefer HLC once component contains 2+ tightly related sub-components

Example:

```tsx
import UserProfile from '@/features/user/components/UserProfile';
```

Never:

```tsx
import Avatar from '@/features/user/components/UserProfile/components/Avatar';
```

---

# Import Rules

## Good

```tsx
import { useAuth } from '@/features/auth/hooks/useAuth';

import UserCard from '@/features/user/components/UserCard';

import UserProfile from '@/features/user/components/UserProfile';

import { Button } from '@/shared/ui/button';
```

## Bad

```tsx
import { useAuth } from '@/features/auth';

import { UserCard } from '@/features/user';

import Avatar from '@/features/user/components/UserProfile/components/Avatar';
```

---

# Component Hierarchy

| Level             | Location                   | Purpose                      |
| ----------------- | -------------------------- | ---------------------------- |
| Shared UI         | shared/ui                  | Pure reusable UI             |
| Shared HLC        | shared/components          | Reusable composed components |
| Feature Component | features/X/components      | Feature-specific UI          |
| Feature HLC       | features/X/components/X    | Complex feature UI           |
| Feature Page      | features/X/FeaturePage.tsx | Route entry                  |

---

# State Management

| State Type   | Tool            |
| ------------ | --------------- |
| Server State | TanStack Query  |
| Client State | Zustand         |
| Form State   | React Hook Form |
| Validation   | Zod             |

Rules:

* Never duplicate server state into Zustand.
* Cache remote data with TanStack Query.
* Keep local UI state close to components.

---

# API Layer

```text
infra/api/
├── user.queries.ts
├── user.mutations.ts
├── order.queries.ts
└── order.mutations.ts
```

Feature hooks consume API definitions.

```tsx
useUser();
useCreateUser();
```

Pages never call APIs directly.

---

# Testing

```text
__tests__/
```

* Hooks
* Business logic
* Utilities

```text
__componentTests__/
```

* Components
* Visual behavior
* User interactions

---

# New Feature Checklist

1. Create feature folder
2. Add types
3. Add schemas
4. Add API definitions
5. Create hooks
6. Create page
7. Create components
8. Add store (if needed)
9. Add tests
10. Register route

Order:

```text
types
→ schemas
→ api
→ hooks
→ page
→ components
→ store
→ tests
→ router
```

---

# Decision Guide

| Need                     | Location           |
| ------------------------ | ------------------ |
| Route Page               | Feature root       |
| Business Component       | Feature components |
| Reusable UI              | shared/ui          |
| Shared Complex Component | shared/components  |
| API Definition           | infra/api          |
| Feature Hook             | feature/hooks      |
| Client State             | feature/stores     |
| Validation               | feature/schemas    |
| Types                    | feature/types      |
| Utility                  | shared/utils       |

---

# Default Stack

* React
* TypeScript
* Vite
* TanStack Query
* Zustand
* React Hook Form
* Zod
* Tailwind CSS
* shadcn/ui
* React Router
* Vitest
* Testing Library
* ESLint
* Prettier
* Husky
* lint-staged

```
```
