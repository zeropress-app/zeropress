# ADR-0005: Client-side Hydration Boundary

## ADR Metadata

* **ADR ID**: ADR-0005
* **Title**: Client-side Hydration Boundary
* **Status**: Accepted
* **Date**: 2025-12-23
* **Decision Owner**: ZeroPress Core Team

---

## Context

ZeroPress delivers user-facing pages as fully static HTML generated at publish time.

However, certain user interactions (search, comments, theme toggles, minor UI affordances) require client-side interactivity.

Without explicit constraints, client-side JavaScript can gradually expand until user pages effectively become applications — contradicting ZeroPress’s core principles defined in:

* ADR-0001: Static Publish for User-Facing Pages
* ADR-0002: Why ZeroPress Is Not a Headless CMS
* ADR-0003: R2 File Layout for Published Content
* ADR-0004: AutoRAG-Based Search Architecture

A clear boundary was required to define **where hydration is allowed and where it is forbidden**.

---

## Decision

ZeroPress enforces a **strict client-side hydration boundary** on user-facing pages.

Hydration is **allowed only for isolated, non-content UI components** and must never:

* Replace or re-render primary content
* Be required to read the page
* Affect SEO-critical elements

Primary content must remain fully readable and meaningful without JavaScript execution.

---

## Hydration Policy

### Allowed (Progressive Enhancement)

The following features may use client-side JavaScript:

* Search input and result rendering
* Comment submission and listing
* Theme (light/dark) toggles
* Copy-to-clipboard buttons
* Minor client-side navigation helpers

These features must:

* Fail gracefully
* Not block initial render
* Not modify published HTML content

---

### Forbidden (Hard Boundary)

The following are explicitly disallowed on user pages:

* Client-side rendering of post/page content
* Hydrating the entire document root
* SPA-style routing between content pages
* Re-fetching content HTML or JSON for display

User pages must never depend on JavaScript to display core content.

---

## Architectural Implications

* JavaScript bundles must be **small, optional, and isolated**
* No global application state on user pages
* No client-side router
* Hydration targets must be explicitly scoped

This ensures that user pages remain documents, not applications.

---

## Rationale

### 1. Failure Tolerance

If JavaScript fails, the page must still function as a readable document.

---

### 2. SEO and AI Compatibility

Search engines and AI systems must consume identical content regardless of JS execution.

---

### 3. Long-Term Maintainability

A strict boundary prevents gradual architectural erosion toward SPA complexity.

---

## Alternatives Considered

### 1. Full SPA Hydration

* **Description**: Hydrate entire page and manage navigation client-side
* **Rejected because**:

  * Violates static-first principle
  * Introduces SEO and reliability risks

### 2. Partial Content Rehydration

* **Description**: Re-render content blocks client-side
* **Rejected because**:

  * Creates dual sources of truth
  * Complicates debugging and indexing

---

## Consequences

### Positive

* User pages remain fast and robust
* JavaScript failures degrade gracefully
* Clear development constraints

### Negative

* Some interactive patterns are intentionally unavailable
* Requires discipline in feature implementation

---

## Impacted Areas

* Front-end UI implementation
* JavaScript bundling strategy
* CSP configuration
* Plugin and extension design

---

## Compatibility With Architecture Principles

This decision reinforces:

* Content as immutable artifacts
* Progressive enhancement over application rendering
* Static-first delivery

as defined in `ARCHITECTURE.md`.

---

## Reconsideration Triggers

This ADR may be revisited if:

* Browser capabilities fundamentally change
* A compelling use-case demands client-side content rendering

Any change must be documented via a superseding ADR.

---

## Related Documents

* ADR-0001: Static Publish for User-Facing Pages
* ADR-0002: Why ZeroPress Is Not a Headless CMS
* ADR-0003: R2 File Layout for Published Content
* ADR-0004: AutoRAG-Based Search Architecture
* ARCHITECTURE.md

---

## Notes

This ADR defines the final guardrail preventing ZeroPress from becoming a client-side application.
