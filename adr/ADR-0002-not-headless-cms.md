# ADR-0002: Why ZeroPress Is Not a Headless CMS

## ADR Metadata

- **ADR ID**: ADR-0002
- **Title**: Why ZeroPress Is Not a Headless CMS
- **Status**: Accepted
- **Date**: 2025-12-23
- **Decision Owner**: ZeroPress Core Team

---

## Context

During early architectural discussions, ZeroPress was frequently compared to existing **Headless CMS** solutions.

Headless CMS architectures typically:

- Store content in a database
- Expose content exclusively via JSON APIs
- Delegate HTML generation to external front-end frameworks

Given ZeroPress’s use of modern infrastructure components (Cloudflare Workers, R2, Pages), adopting a Headless CMS model was initially considered a natural fit.

However, deeper analysis revealed a fundamental mismatch between the **philosophical assumptions of Headless CMS** and the **core goals of ZeroPress**.

---

## Decision

ZeroPress **explicitly does not adopt a Headless CMS architecture**.

Instead, ZeroPress is designed as a:

> **Static-first, file-native CMS where published content exists as final HTML artifacts**.

APIs are used only to support content creation, publishing, indexing, and interaction — not to deliver primary page content to users.

---

## Rationale

### 1. Content Should Exist as Artifacts, Not API Responses

In Headless CMS architectures, content existence is coupled to API availability.

In ZeroPress:

- Published content exists as **files** (HTML, JSON, XML)
- These files are the authoritative delivery format for user-facing pages.
- APIs are not required for content access

This guarantees content survivability regardless of backend runtime availability.

---

### 2. User Pages Must Be Documents, Not Applications

Headless CMS models implicitly convert every site into a web application.

This introduces:

- JavaScript execution dependency
- Hydration complexity
- Failure modes resulting in blank or partial pages

ZeroPress treats user-facing pages as **documents**:

- Fully rendered at publish time
- Immediately readable by browsers, crawlers, and AI systems
- Independent of JavaScript for core content

---

### 3. Deterministic SEO and Indexability

Headless CMS SEO relies on secondary strategies such as SSR, SSG, or ISR.

These approaches introduce:

- Operational complexity
- Build-time or request-time coupling
- Non-deterministic crawler behavior

ZeroPress delivers **complete HTML at rest**, ensuring:

- Predictable crawler indexing
- Direct compatibility with search engines
- Seamless integration with file-crawling systems such as AutoRAG

---

### 4. Clear Separation of Responsibilities

ZeroPress enforces strict system boundaries:

- **Admin System**: Application-driven, API-centric, authenticated
- **User Pages**: Static, anonymous, cacheable

Headless CMS architectures blur this boundary by making user pages dependent on runtime systems.

---

## Alternatives Considered

### 1. Traditional Headless CMS

- **Description**: JSON APIs as the primary content delivery mechanism
- **Rejected because**:
  - Content availability tied to API runtime
  - Increased operational surface
  - User pages become application-dependent

### 2. Hybrid Headless + SSG

- **Description**: API-driven content with pre-rendered HTML via frameworks
- **Rejected because**:
  - Framework dependency lock-in
  - Build scalability challenges
  - Redundant content transformation layers

---

## Consequences

### Positive

- Content survives system outages
- Pages load without JavaScript dependencies
- SEO and AI indexing are deterministic
- Infrastructure cost and complexity are minimized

### Negative

- Real-time content mutation is not supported
- Per-user personalization on content pages is limited
- Content updates require republishing

---

## Impacted Areas

- Overall system architecture
- Content publishing pipeline
- Search and indexing strategy
- Client-side rendering boundaries

---

## Compatibility With Architecture Principles

This decision reinforces the principles defined in `ARCHITECTURE.md`:

- Published content exists as files
- User pages are static and immutable until republished
- APIs are supporting components, not delivery mechanisms

---

## Reconsideration Triggers

This ADR may be revisited if:

- ZeroPress shifts toward runtime-personalized content delivery
- File-based publishing becomes a scalability bottleneck
- Platform constraints invalidate static-first assumptions

Any reversal must be documented in a new ADR that supersedes this decision.

---

## Related Documents

- ADR-0001: Static Publish for User-Facing Pages
- ARCHITECTURE.md

---

## Notes

This ADR defines ZeroPress’s philosophical boundary against Headless CMS architectures.
It serves as a guiding constraint for all future design decisions.

