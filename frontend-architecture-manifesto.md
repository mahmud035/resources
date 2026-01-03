Perfect. What you’re asking for is **exactly what senior engineers do**: they codify their thinking into a *manifesto* so future decisions are automatic, not emotional.

Below is a **1-page Frontend Architecture Manifesto** written in the same **directive, enforceable style** as your backend rules.
You can paste this directly into:

* ChatGPT Custom Instructions
* Claude Custom Instructions
* Your personal engineering playbook
* Project README “Architecture” section

No fluff. No ambiguity. This is a **contract with yourself**.

---

# Frontend Architecture Manifesto

### (React • TypeScript • TanStack Query • Tailwind CSS)

## Purpose

This architecture exists to build **scalable, readable, deadline-resistant frontend applications** that align with real backend systems and remain maintainable under pressure.

This is not a learning architecture.
This is a **production standard**.

---

## Core Rules (Non-Negotiable)

### 1. Feature-Driven Structure Only

**Never build component soup.**

All domain logic lives inside `features/`.

Each feature maps **1:1 to a backend concept**.

Examples:

* `departments`
* `faculties`
* `products`
* `orders`
* `users`
* `search`

A feature owns:

* Its API calls
* Its TypeScript types
* Its domain UI
* Its internal logic

**No cross-feature imports unless explicitly shared.**

---

### 2. Pages Orchestrate. Features Execute.

**Pages (`/pages`)**

* Handle routing
* Read route params
* Compose features
* Control layout flow

Pages must **never**:

* Fetch raw data directly
* Contain business logic
* Transform API responses

**Features do the work. Pages decide placement.**

---

### 3. API-First Thinking

The frontend mirrors the backend.

* API response shapes define frontend types
* No “UI-only” fake models
* All server state flows through TanStack Query

All API calls:

* Live inside feature folders
* Use a shared Axios instance
* Return typed responses

```ts
ApiResponse<T> is the contract.
```

If the backend changes, the frontend should **fail fast** at compile time.

---

### 4. Server State Belongs to TanStack Query

TanStack Query owns:

* Data fetching
* Caching
* Loading states
* Error states
* Background refetching

Local `useState` is allowed **only** for:

* UI toggles
* Form inputs
* Modal state
* Temporary UI behavior

No duplicated server state.
No manual cache syncing.

---

### 5. UI Components Are Dumb by Design

Reusable UI components live in `components/ui/`.

They:

* Receive props
* Render UI
* Emit events

They must **never**:

* Fetch data
* Know about APIs
* Know about routing
* Contain business rules

If a component needs logic, it becomes a **feature component**, not a UI primitive.

---

### 6. Hooks Extract Reusable Logic Only

Hooks (`/hooks`) exist to:

* Encapsulate side effects
* Share repeated logic
* Improve readability

Hooks must be:

* Small
* Focused
* UI-agnostic

No abstraction for abstraction’s sake.
If logic isn’t reused, it stays local.

---

### 7. Styling Philosophy

* Tailwind CSS v4
* University / enterprise-grade minimalism
* Clear hierarchy over decoration
* Accessibility by default
* Skeleton loaders over spinners
* Explicit microcopy for external actions

The UI must **feel trustworthy**, not flashy.

---

### 8. Error, Empty, and Loading States Are Mandatory

Every data-driven view must define:

* Loading state
* Empty state
* Error state

Silence is a bug.
Ambiguity is a UX failure.

---

### 9. Predictability Over Cleverness

This architecture favors:

* Readability over terseness
* Convention over creativity
* Explicitness over magic

Code should be **boring to read** and **easy to reason about**.

---

## Standard Frontend Project Structure

```
src/
├── api/                # Axios instance, shared API types
├── assets/             # Logos, images
├── components/
│   ├── layout/         # Navbar, Footer, Layout shells
│   └── ui/             # Buttons, Inputs, Cards, Skeletons
├── features/           # Domain-driven modules
├── hooks/              # Reusable logic
├── pages/              # Route-level orchestration
├── providers/          # QueryProvider, context providers
├── routes/             # Router config
├── utils/              # Pure helpers
├── styles/             # Tailwind entry
└── main.tsx
```

---

## Final Rule

If a decision breaks these principles, **do not ship it**.

This architecture exists so:

* Deadlines don’t force shortcuts
* Refactors don’t cause fear
* New features don’t cause regressions
* Future you doesn’t curse present you

**This is the standard. Follow it strictly.**


