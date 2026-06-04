---
name: frontend-architecture
description: Framework- and tool-agnostic frontend architecture pattern engine. Scans the project first, then applies a uniform domain-centric structure where every domain lives in modules/<domain>/ (service, hook, store, types, utils). The detected state/data tool changes file contents, not file location. Use when scaffolding, placing, or refactoring FE code, or when the user attaches /frontend-architecture.
version: 0.1.0
disable-model-invocation: true
---

# Frontend Architecture

A pattern engine for organizing frontend code consistently across any framework and toolset. It does not assume Next.js, React, Chakra, or any library. It scans the open workspace, infers conventions, and applies one uniform architecture.

## When to use

- User attaches this skill or mentions `frontend-architecture`.
- Scaffolding a new feature, placing new files, or refactoring structure.
- Reviewing whether code lives in the right place.

Always **adapt to the open workspace**. Detect the framework, tools, language, and conventions before creating anything. Never assume a stack the scan did not confirm, and never add dependencies without explicit approval.

## 1. Discovery first (mandatory)

Before proposing or creating any files, scan and summarize.

Scan targets:

1. Root config: `package.json` (deps + scripts), `tsconfig.json` / `jsconfig.json` (paths), workspace files (`pnpm-workspace.yaml`, turbo/nx).
2. Source layout: top-level source folders, route folder(s), `modules/` (or equivalent), test folders.
3. Conventions: file suffixes (`*.service.*`, `*.hook.*`, `*.store.*`, `*.slice.*`, `*.types.*`, `*.api.*`), import alias.
4. Tools: framework/router, state library, data-fetching library, i18n, test runner, lint/format.
5. Language: TypeScript vs JavaScript (drives `.ts/.tsx` vs `.js/.jsx` and whether `*.types` files apply).

Output a short **discovery summary** before scaffolding:

- Framework + router detected
- State / data / i18n / testing tools detected
- Existing folder + naming conventions
- Proposed module mapping (reuse vs add)
- Confidence: High / Medium / Low

If confidence is Low (new/empty repo or conflicting patterns), ask the clarification questions in section 8 before writing files.

## 2. Architecture model (agnostic layers)

Map these conceptual layers to the project's real paths; do not force folder names that conflict with the repo.

| Layer | Holds |
|-------|-------|
| routes | Pages / views / route handlers (framework-native location) |
| components | UI primitives + feature compositions |
| modules | Self-contained domain logic (service, hook, store, types, utils) |
| `hooks/consolidated/` | Single-domain composites of `modules/<domain>` hooks; export `useConsolidated<Domain>` |
| `hooks/utility/` | Domain-agnostic reusable hooks; export `use<Thing>` |
| `config/` | Runtime wiring: HTTP/client instances, query client, cookies, env-driven setup, SDK bootstraps |
| `constants/` | Static, environment-independent values: keys, enums, route paths, defaults, durations |
| utils | Pure, domain-agnostic helpers |
| tests | Follow the project's runner and placement |

There is **no separate `state/`, `stores/`, or `features/` layer**. All domain state lives inside the domain's module folder. Individual domain hooks live in `modules/<domain>/<domain>.hook.*`; the `hooks/` layer only composes them (consolidated) or holds generic reusable hooks (utility).

## 3. Domain module structure (core rule)

Every domain is one folder: `modules/<domain>/`. All logic for that domain lives inside it.

```text
modules/<domain>/
├── <domain>.service.{ts|js}    # API / network calls, endpoint map
├── <domain>.hook.{ts|js}       # data/query hooks, orchestration for the domain
├── <domain>.store.{ts|js}      # client/global state for the domain (only when needed)
├── <domain>.types.{ts|d.ts}    # request/response + domain types (TS projects)
├── <domain>.utils.{ts|js}      # domain-only pure helpers (optional)
└── index.{ts|js}               # barrel, when widely imported
```

Example — `order` domain:

```text
modules/order/
├── order.service.ts
├── order.hook.ts
├── order.store.ts
├── order.types.ts
└── order.utils.ts
```

Rules:

- Never split a domain's state into a global folder; it stays in `modules/<domain>/`.
- Keep file suffixes consistent across all domains in the repo.
- TypeScript: `.ts/.tsx` + `<domain>.types.ts`. JavaScript: `.js/.jsx`, no `.types` file.
- If the repo already uses a variant suffix (e.g. `*.api.ts` instead of `*.service.ts`), match it.

### Consolidated & utility hooks

The `hooks/` layer has two folders. Individual domain hooks stay in `modules/<domain>/<domain>.hook.*`; this layer never owns domain logic.

```text
src/hooks/
├── consolidated/   # use-consolidated-<domain>.{ts|js} -> useConsolidated<Domain>
└── utility/        # use-<thing>.{ts|js} -> use<Thing>
```

- **`hooks/consolidated/`** — combines several hooks from **one** domain into a single hook. It imports from `modules/<domain>/<domain>.hook.*` and composes them; it does not redefine domain logic. File `use-consolidated-<domain>.{ts|js}`, export `useConsolidated<Domain>` (e.g. `useConsolidatedOrder`).
- **`hooks/utility/`** — domain-agnostic reusable hooks (`useCountdown`, `useMouseMove`, `useDebounce`). File `use-<thing>.{ts|js}`, export `use<Thing>`.
- File names are kebab and mirror the export; extensions follow detected TS/JS.

```typescript
// hooks/consolidated/use-consolidated-order.ts — composes order-domain hooks
import { useOrderId, useOrderDetails, useCartCount } from "{srcAlias}/modules/order/order.hook";

export const useConsolidatedOrder = () => {
  const orderId = useOrderId();
  const orderDetails = useOrderDetails();
  const cartCount = useCartCount();
  return { orderId, orderDetails, cartCount };
};
```

## 4. Inference matrix (tool decides contents, not location)

The detected state/data tool changes **what** goes in `<domain>.store.*` / `<domain>.hook.*`, never **where**. Precedence: existing repo convention > this matrix > minimal default.

Services (uniform): all API calls in `modules/<domain>/<domain>.service.*`; domain types alongside.

State, in `<domain>.store.*`:

| Detected tool | Contents |
|---------------|----------|
| Zustand | `create` store + selectors |
| RTK | `createSlice` + selectors; register the slice in the existing root store (do not relocate it) |
| MobX | observable store class |
| Jotai / Recoil / atomic | domain atoms grouped in the store file |
| None | do not add global state; keep state local; ask before introducing a library |

Data fetching, in `<domain>.hook.*`:

| Detected tool | Contents |
|---------------|----------|
| React Query / SWR | query/mutation hooks + colocated keys |
| RTK Query | endpoints injected into the domain's API; hooks re-exported from the domain |
| None | thin hooks wrapping the service + local state |

i18n / routing:

- i18n: follow the existing setup. If none exists, skip i18n files unless the user requests multilingual.
- Routing: use the framework's native route location. Never force `app/[lng]`, route groups, or `"use client"` unless the repo already uses them.

### Config vs constants

- `config/` holds runtime wiring — anything that creates an instance, reads env vars, or bootstraps a library (HTTP client, query/cache client, cookie adapters, SDK setup). Side effects and `"use client"` are allowed here.
- `constants/` holds static values — fixed data with no runtime behavior (storage keys, route paths, enums, default sizes, time durations). Prefer `as const` objects and derive types with `typeof`.
- Rule of thumb: instantiates or depends on the environment at runtime -> `config/`; fixed value known ahead of time -> `constants/`.
- Match the repo's existing naming: `config/<name>.{ts|js}` (e.g. `http-client.ts`), `constants/<name>.const.{ts|js}` (e.g. `route-path.const.ts`).
- Create either folder only when the project needs it (minimal-delta).

## 5. Naming and placement

- **UI prefix:** detect the prefix from existing primitives in the project's `ui` folder; otherwise derive from `package.json` `name` (strip scope; kebab for files, Pascal for components). See [reference.md](reference.md#ui-prefix).
- **Path alias:** read `compilerOptions.paths` for the alias that maps to the source root (e.g. `@/`, `@source/`, `~`). Use it in imports; do not invent one the project does not define.
- New reusable primitives go in the UI layer, not feature folders. Features compose primitives.

## 6. Minimal-delta scaffolding

- Create only the files/folders the task and inferred stack require.
- Within a domain, create only the files that domain uses (omit `<domain>.store.*` when there is no domain state).
- Reuse naming/suffix patterns from existing domains.
- On conflicting conventions, choose the dominant one and note the tie-break.
- Do not move or rename broad areas unless the user explicitly requests a refactor.

## 7. Adaptive workflows

### New feature

1. Run discovery; infer the domain folder, suffixes, and language.
2. Confirm ambiguities (section 8) if confidence is Low.
3. Add/update the route or view in the framework-native location.
4. Create/extend `modules/<domain>/`:
   - `<domain>.service.*` for API calls
   - `<domain>.hook.*` for query/orchestration hooks
   - `<domain>.store.*` only if the domain needs client/global state
   - `<domain>.types.*` for types (TS)
5. Wire UI; reuse existing primitives.
6. If using RTK, register the domain slice in the existing root store.
7. If the domain exposes several related hooks consumed together, add `hooks/consolidated/use-consolidated-<domain>.*` that composes them.
8. Add tests with the project's runner and placement.
9. Run the project's existing lint/test scripts.

### UI-only change

1. Reuse existing primitives and patterns.
2. Do not add domain modules or state unless required.

### Refactor

1. Confirm scope and constraints (section 8).
2. Propose a migration map before moving files (e.g. `features/order` + `stores/order` -> `modules/order/`).
3. Execute in phases; keep imports and tests green at each boundary.

## 8. Clarification questions

Ask when the repo is new/nearly empty, patterns conflict, a whole-project refactor is requested, or new tooling would be required.

**New / empty project:**

1. Which framework and router?
2. Which state approach for domain stores (Zustand, RTK, MobX, atomic, none)?
3. Which data-fetching approach (React Query, RTK Query, SWR, none)?
4. TypeScript or JavaScript?
5. i18n now? If yes, which library and which locales?
6. Which test runner, and centralized vs colocated tests?
7. Confirm the domain convention: `modules/<domain>/<domain>.{service,hook,store,types}.*`?

**Whole-project refactor:**

1. Scope: full repo or selected domains?
2. Moving/renaming files allowed, or incremental only?
3. May existing `state/` `stores/` `features/` folders collapse into `modules/<domain>/`?
4. Are dependency changes allowed?
5. Any non-negotiable conventions to keep?
6. Preserve old import paths via barrels/shims during migration?
7. Rollout: big-bang or phased per domain?

**Tie-breakers (single question each):** `features/` vs `modules/` as primary; Jest vs Vitest; which `src` alias is canonical; `*.api.*` vs `*.service.*`.

Full question sets and a migration example: [reference.md](reference.md#clarification-questions).

## Guardrails

- All domain logic lives in `modules/<domain>/` — never a separate state/stores/features layer.
- The state/data tool changes file contents, not file location.
- Consolidated hooks only compose existing module hooks; they must not redefine domain logic.
- Consolidated hooks are single-domain; if a cross-domain composite seems warranted, ask the user before creating one.
- Never assume a framework or tool the scan did not confirm.
- Never add dependencies without explicit approval.
- Prefer existing repo conventions over theoretical ideals.

## Additional resources

Extended detection rules, inference matrix, examples, and checklists: [reference.md](reference.md)
