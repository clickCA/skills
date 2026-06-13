---
name: react-feature-architecture
description: >
  Guidance for scaffolding and reviewing LLM-friendly, feature-based React
  applications. Use this skill whenever the user asks about React project
  structure, folder organisation, feature-based architecture, or how to
  organise a React codebase for better AI-assisted development. Also trigger
  when the user asks to scaffold a new feature, review an existing structure,
  or wants to know the "right" way to split components, hooks, and API layers.
  Apply even when the request is phrased casually — e.g. "how should I
  organise my React app", "where do I put my fetch logic", "how do I structure
  a feature folder".
---

# React LLM-friendly feature-based architecture

A pattern for React apps that keeps every feature self-contained and small
enough to fit in a single LLM context window — making AI-assisted development
faster and more accurate.

---

## Core idea

Group code by **feature**, not by type. Every file a feature needs lives in
one folder. A feature is anything a user can *do* (auth, checkout, dashboard).

```
src/
├── features/          # one folder per user-facing feature
│   ├── auth/
│   ├── products/
│   └── checkout/
├── shared/            # zero feature imports — pure utilities + UI atoms
│   ├── ui/
│   ├── hooks/
│   └── lib/
└── app/               # router, global store, providers
    ├── router.tsx
    └── store.ts
```

---

## Feature folder anatomy

Every feature follows this exact file layout. Introduce files only when needed;
start with just `Page`, `useFeature`, and `api`:

```
features/auth/
├── AuthPage.tsx        # entry point — composes only, no logic
├── LoginForm.tsx       # presentational — props in, JSX out
├── useAuth.ts          # all state + side-effects for this feature
├── auth.api.ts         # raw network calls, no React imports
├── auth.types.ts       # TypeScript types — the feature's vocabulary
└── auth.test.ts        # colocated tests
```

### File roles

| File | Purpose | Rules |
|------|---------|-------|
| `FeaturePage.tsx` | Compose hooks + components | No fetch, no business logic. Target ≤ 80 lines. |
| `FeatureForm.tsx` / `FeatureList.tsx` | Presentational components | Props only. No hooks except `useState` for local UI. |
| `useFeature.ts` | State, effects, derived values | Returns a stable typed API. Declare `@returns` JSDoc. |
| `feature.api.ts` | Network layer | No React imports. Pure: args in → typed response out. |
| `feature.types.ts` | TypeScript types | Attach to every LLM prompt about this feature. |
| `feature.test.ts` | Tests | Colocated, not in a separate `__tests__/` folder. |

---

## Dependency rules (hard boundaries)

```
app/  →  features/  →  shared/
                  ↑
     features may NOT import each other
```

- `shared/` never imports from `features/`
- Features never import from other features — communicate through `app/store.ts` or URL state
- `app/` wires features together via the router and global store

---

## File naming conventions

Follow these suffixes consistently — LLMs infer context from filenames:

| Suffix | Example | Meaning |
|--------|---------|---------|
| `.page.tsx` | `AuthPage.tsx` | Feature entry point |
| `.api.ts` | `auth.api.ts` | Network / data-fetching layer |
| `.types.ts` | `auth.types.ts` | TypeScript types |
| `.test.ts` | `auth.test.ts` | Tests |
| `.store.ts` | `ui.store.ts` | Feature-scoped Zustand slice |
| (no suffix) | `LoginForm.tsx` | Component |
| `use*.ts` | `useAuth.ts` | Custom hook |

---

## Data flow

```
Router
  └─ FeaturePage.tsx          (compose only)
        ├─ useFeature.ts       (state + effects)
        │     ├─ store.ts      (global state — session, theme)
        │     └─ feature.api.ts → REST / tRPC / GraphQL
        └─ FeatureForm.tsx     (presentational)
```

---

## LLM-friendly conventions

### 1. Keep files small
Target ≤ 150 lines per file. Smaller files fit entirely in one context window,
producing more accurate suggestions. If a file grows beyond this, split by
extracting a child component or a second hook.

### 2. Explicit return types on hooks
```ts
// ✅ Good — LLM sees the full contract at a glance
export function useAuth(): {
  user: User | null;
  login: (creds: Credentials) => Promise<void>;
  logout: () => void;
  status: 'idle' | 'loading' | 'error';
} { … }

// ❌ Bad — LLM must infer return shape from implementation
export function useAuth() { … }
```

### 3. Types file as context attachment
`auth.types.ts` gives an LLM the full domain vocabulary in ~30 lines. Always
attach it when prompting about a feature.

```ts
// auth.types.ts — attach to every LLM prompt about auth
export interface User { id: string; email: string; role: 'admin' | 'viewer'; }
export interface Credentials { email: string; password: string; }
export type AuthStatus = 'idle' | 'loading' | 'authenticated' | 'error';
```

### 4. Pure API functions
```ts
// auth.api.ts — no React, no side effects
export async function loginUser(creds: Credentials): Promise<User> {
  const res = await fetch('/api/auth/login', {
    method: 'POST',
    body: JSON.stringify(creds),
  });
  if (!res.ok) throw new ApiError(res.status);
  return res.json();
}
```

Pure functions with typed signatures generate correct tests with zero extra
prompting. Mark the file with `// generated-safe` if it's fully regenerable.

### 5. Named parameters on hooks
```ts
// ✅ Named params survive refactors without breaking call sites
useProducts({ page: 1, filter: 'active' })

// ❌ Positional args break when a new param is inserted
useProducts(1, 'active')
```

### 6. One export per file
Default export = the main thing. Named exports = types only.
LLMs should never have to guess which export is primary.

### 7. Sparse barrel files
Only export at the feature boundary (`features/auth/index.ts`). Avoid deep
barrel chains — they obscure import origins.

```ts
// features/auth/index.ts — only what shared code needs
export type { User, AuthStatus } from './auth.types';
export { useAuth } from './useAuth';
```

### 8. Pin dependency versions in comments
```ts
// queryClient.ts — TanStack Query v5
// Upgrade guide: https://tanstack.com/query/v5/docs/react/guides/migrating-to-v5
```
Prevents stale completions from models trained on older API shapes.

---

## Scaffolding a new feature

When asked to scaffold a feature, produce files in this order:

1. `feature.types.ts` — define the domain types first
2. `feature.api.ts` — typed network functions
3. `useFeature.ts` — hook with explicit return type
4. `FeaturePage.tsx` — thin composition page
5. `feature.test.ts` — at minimum, tests for the hook

Ask the user for:
- Feature name (used to derive all filenames)
- Data shape / API endpoint
- Key user actions (login, submit, delete…)
- Global state needed (if any)

---

## Reviewing an existing structure

When asked to review a React codebase structure, check for:

- [ ] Files grouped by feature, not by type (`components/`, `hooks/` at root = red flag)
- [ ] Feature folders contain their own `*.api.ts` and `*.types.ts`
- [ ] No cross-feature imports
- [ ] Page components under 80 lines
- [ ] Hook return types are explicit
- [ ] Tests colocated next to source
- [ ] `shared/` has zero feature imports
- [ ] Barrel files exist only at feature boundary

Report findings as a short table: file / issue / recommended fix.

---

## Technology recommendations

These pair well with this architecture but are not required:

| Concern | Recommended | Why LLM-friendly |
|---------|-------------|-----------------|
| Server state | TanStack Query v5 | Key factory pattern is easy to generate |
| Client state | Zustand | Tiny API, one-file slices |
| Routing | React Router v6 | Declarative, easy to scaffold |
| Types | TypeScript strict mode | Explicit types = better completions |
| Testing | Vitest + Testing Library | Colocated, fast, minimal config |
| Forms | React Hook Form | Uncontrolled — less state to explain to an LLM |

---

## Quick reference: what to attach to LLM prompts

| Task | Files to attach |
|------|----------------|
| Add a field to a form | `FeatureForm.tsx` + `feature.types.ts` |
| Fix a bug in data fetching | `useFeature.ts` + `feature.api.ts` + `feature.types.ts` |
| Write tests for the hook | `useFeature.ts` + `feature.types.ts` + `feature.test.ts` |
| Add a new route | `router.tsx` + `FeaturePage.tsx` |
| Debug global state | `store.ts` + `useFeature.ts` |
| Regenerate API layer | `feature.api.ts` + `feature.types.ts` |
