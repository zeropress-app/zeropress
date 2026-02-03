# ADR-0001: Static Publish for User-Facing Pages

## ADR Metadata

* **ADR ID**: ADR-0001
* **Title**: Static Publish for User-Facing Pages
* **Status**: Accepted
* **Date**: 2025-12-23
* **Decision Owner**: ZeroPress Core Team

---

## Context

ZeroPress aims to be a modern CMS that prioritizes performance, reliability, security, and long-term maintainability.

Early design discussions considered multiple rendering strategies for user-facing pages, including:

* Full SPA rendering using a single `index.html`
* Request-time server-side rendering (SSR) using Cloudflare Workers
* Static site generation (SSG) using a framework such as Next.js

Key constraints and goals included:

* Excellent SEO without reliance on JavaScript execution
* Minimal operational complexity
* Clear separation between Admin and User-facing systems
* Compatibility with file-based content storage (Cloudflare R2)
* Graceful degradation in failure scenarios

---

## Decision

ZeroPress will generate **fully static HTML pages at publish time** for all user-facing content and serve them via **Cloudflare Pages**.

User-facing pages will not be rendered dynamically at request time.

---

## Rationale

This decision aligns with the core architectural principles of ZeroPress:

* **Static HTML provides the highest reliability**: pages remain accessible even if Workers, APIs, or JavaScript fail
* **SEO is deterministic**: crawlers receive complete HTML without delayed or partial rendering
* **Operational simplicity**: no runtime rendering infrastructure is required
* **Security posture is improved**: attack surface is significantly reduced
* **Cost predictability**: static delivery scales without per-request compute cost

Static publish also enables a clear mental model:

> Published content exists as files, not as database rows or runtime responses.

---

## Alternatives Considered

### 1. Full SPA Rendering

* **Description**: Serve a single `index.html` and render content client-side by fetching JSON data
* **Rejected because**:

  * SEO becomes non-deterministic
  * JavaScript failure results in a blank page
  * User-facing pages become application-dependent rather than content-dependent

### 2. Request-Time SSR via Cloudflare Workers

* **Description**: Render HTML on each request using Workers
* **Rejected because**:

  * Increased latency and complexity
  * Higher operational risk and cost
  * Blurs the separation between Admin and User systems

### 3. Framework-Based SSG (e.g., Next.js)

* **Description**: Pre-generate HTML via a Node.js-based build pipeline
* **Rejected because**:

  * Heavy framework dependency
  * Build-time scaling issues with large content sets
  * Conflicts with ZeroPress’s lightweight, infrastructure-native approach

---

## Consequences

### Positive

* User pages are extremely fast and cacheable
* Pages remain accessible during partial system outages
* SEO behavior is predictable and robust
* Clear boundary between content publishing and content delivery

### Negative

* Content updates are not reflected until republished
* Highly dynamic per-user personalization is not supported
* Search indexing may lag due to periodic crawling

---

## Impacted Areas

* Content publishing pipeline
* R2 storage layout
* Cloudflare Pages configuration
* Search architecture (AutoRAG crawling published files)
* Client-side navigation strategy

---

## Compatibility With Architecture Principles

This decision directly supports the principles defined in `ARCHITECTURE.md`:

* User pages are fully static
* Published content exists as files
* Failure degrades gracefully to static HTML
* Admin and User systems remain strictly separated

---

## Reconsideration Triggers

This decision may be revisited if:

* Static HTML generation becomes a scalability bottleneck
* A compelling use-case requires request-time rendering
* Cloudflare platform capabilities fundamentally change

Any change must be documented in a new ADR that supersedes this record.

---

## Related Documents

* ARCHITECTURE.md
* ADR-0002 (planned): Search Architecture with AutoRAG

---

## Notes

This ADR establishes the foundational rendering model for ZeroPress.
All future architectural decisions must be evaluated against this baseline.
