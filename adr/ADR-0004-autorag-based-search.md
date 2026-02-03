# ADR-0004: AutoRAG-Based Search Architecture

## ADR Metadata

* **ADR ID**: ADR-0004
* **Title**: AutoRAG-Based Search Architecture
* **Status**: Accepted
* **Date**: 2025-12-23
* **Decision Owner**: ZeroPress Core Team

---

## Context

ZeroPress delivers user-facing content as static files stored in Cloudflare R2 and served via Cloudflare Pages.

Search is a core feature but must not violate the static-first principle established in earlier ADRs:

* ADR-0001: Static Publish for User-Facing Pages
* ADR-0002: Why ZeroPress Is Not a Headless CMS
* ADR-0003: R2 File Layout for Published Content

Traditional search architectures (database-backed full-text search, request-time indexing, framework-level search pages) introduce runtime coupling and operational complexity that conflict with ZeroPress’s goals.

Cloudflare **AutoRAG (Beta)** provides a file-crawling, AI-assisted retrieval mechanism that aligns with ZeroPress’s file-native approach.

---

## Decision

ZeroPress will implement search using **Cloudflare AutoRAG**, configured to:

* Periodically crawl **published files in R2**
* Index HTML and `meta.json` content
* Provide AI-assisted natural language retrieval

Search query handling will be executed via **Cloudflare Workers**, which act as a thin orchestration layer between the user interface and AutoRAG.

Search result pages themselves are not statically generated.

---

## Architecture Overview

```
User Browser
   ↓ query
Search UI (static)
   ↓
Cloudflare Worker (search endpoint)
   ↓
AutoRAG Index (R2 crawler)
   ↓
Search Results (references to static pages)
```

---

## Rationale

### 1. File-Native Indexing

AutoRAG indexes **the same artifacts that users consume**:

* `index.html`
* `meta.json`

This guarantees:

* No divergence between search index and published content
* No need for secondary data pipelines
* Immediate compatibility with static publishing

---

### 2. Zero Runtime Coupling for Content Pages

Search queries are dynamic, but **content pages remain static**.

* A failure in AutoRAG or Workers does not affect page availability
* Search is an enhancement, not a dependency

---

### 3. AI-Native Retrieval Without Application Lock-in

AutoRAG enables:

* Semantic and natural-language queries
* Context-aware result ranking

without requiring:

* Application-level embeddings
* Vector database management
* Custom AI infrastructure

---

### 4. Minimal Surface Area

Workers are limited to:

* Receiving search queries
* Forwarding requests to AutoRAG
* Returning structured results

They do not:

* Render content
* Transform HTML
* Maintain state

---

## Alternatives Considered

### 1. Database-Backed Full-Text Search

* **Description**: Store content in a database and index via SQL or search engines
* **Rejected because**:

  * Violates file-native storage model
  * Introduces data duplication
  * Requires runtime availability

### 2. Framework-Level Search (Next.js, Nuxt)

* **Description**: Search pages generated or hydrated by front-end frameworks
* **Rejected because**:

  * Framework lock-in
  * Increased bundle size and hydration scope
  * Conflicts with static-first delivery

### 3. Client-Side Search Index

* **Description**: Ship a prebuilt index to the browser
* **Rejected because**:

  * Poor scalability
  * Large payload sizes
  * Limited semantic capabilities

---

## Consequences

### Positive

* Search remains decoupled from content delivery
* Index reflects published state automatically
* AI search capability with minimal operational overhead
* No impact on page performance or SEO

### Negative

* Search index freshness depends on crawl schedule
* AutoRAG is currently a beta service
* Search results cannot be fully pre-rendered

---

## Impacted Areas

* Search UI implementation
* Worker routing and permissions
* R2 publish and update lifecycle
* Monitoring and observability

---

## Compatibility With Architecture Principles

This decision reinforces:

* Static-first content delivery
* File-native indexing
* Runtime isolation between content and interaction layers

as defined in `ARCHITECTURE.md`.

---

## Reconsideration Triggers

This ADR may be revisited if:

* AutoRAG exits beta with incompatible constraints
* File-based crawling becomes insufficient at scale
* Deterministic, non-AI search becomes a strict requirement

Any change must be documented in a superseding ADR.

---

## Related Documents

* ADR-0001: Static Publish for User-Facing Pages
* ADR-0002: Why ZeroPress Is Not a Headless CMS
* ADR-0003: R2 File Layout for Published Content
* ARCHITECTURE.md

---

## Notes

This ADR establishes search as a **supporting, non-critical system**.
Content must remain accessible and authoritative without search availability.
