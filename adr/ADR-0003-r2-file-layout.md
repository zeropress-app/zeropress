# ADR-0003: R2 File Layout for Published Content

## ADR Metadata

* **ADR ID**: ADR-0003
* **Title**: R2 File Layout for Published Content
* **Status**: Accepted
* **Date**: 2025-12-23
* **Decision Owner**: ZeroPress Core Team

---

## Context

ZeroPress publishes user-facing content as static files stored in **Cloudflare R2** and served via **Cloudflare Pages**.

Given this static-first architecture, the **file layout within R2** becomes a core part of the system contract:

* It defines how content is served
* How search systems crawl and index content
* How tooling and future migrations interact with published data

The content model includes posts and pages with metadata such as slug, publish status, timestamps, and content format.

A clear, predictable, and future-proof file layout was required.

---

## Decision

ZeroPress will store published content in R2 using a **path-based, content-type-oriented file layout**.

Each published URL corresponds directly to a directory containing:

* A fully rendered `index.html`
* A machine-readable `meta.json`
* Optional auxiliary files (e.g., feeds, structured data)

The URL structure and file structure are intentionally isomorphic.

---

## File Layout Specification

### Post Example

```
/posts/direct-connect-사용하는-이유-aws-direct-connect-구성-방법/
  ├─ index.html
  └─ meta.json
```

### Page Example

```
/pages/about/
  ├─ index.html
  └─ meta.json
```

### Sitemap and Feeds

```
/sitemap.xml
/rss.xml
```

---

## meta.json Structure

`meta.json` is designed to be:

* Stable
* Forward-compatible
* Search- and AI-friendly

Example:

```
{
  "id": 9,
  "title": "Direct Connect 사용하는 이유, AWS Direct Connect 구성 방법",
  "slug": "direct-connect-사용하는-이유-aws-direct-connect-구성-방법",
  "type": "post",
  "contentFormat": "html",
  "createdAt": "2025-10-26T18:42:11.000Z",
  "updatedAt": "2025-10-28T00:51:08.000Z",
  "published": true,
  "canonicalUrl": "/posts/direct-connect-사용하는-이유-aws-direct-connect-구성-방법/"
}
```

---

## Rationale

### 1. URL-to-File Isomorphism

Each public URL maps directly to a file path in R2.

This:

* Simplifies debugging and reasoning
* Eliminates routing ambiguity
* Enables trivial static hosting

---

### 2. HTML as Primary Artifact

`index.html` is the definitive user-facing representation.

* No runtime rendering is required
* Browsers, crawlers, and AI systems consume the same artifact

---

### 3. Machine-Readable Metadata Separation

`meta.json` exists alongside HTML to support:

* Search indexing (AutoRAG crawling)
* Tooling and analytics
* Future features without HTML parsing

This avoids coupling metadata extraction to HTML structure.

---

### 4. Minimal and Predictable Layout

The layout intentionally avoids:

* Hash-based filenames
* Build-system-specific artifacts
* Framework-specific directory conventions

This ensures long-term durability and easy migration.

---

## Alternatives Considered

### 1. Flat File Layout

* **Description**: All files stored at root with encoded filenames
* **Rejected because**:

  * Poor human readability
  * Difficult URL mapping
  * Harder incremental updates

### 2. API-First JSON Storage

* **Description**: Store only JSON and render HTML dynamically
* **Rejected because**:

  * Violates static-first principle
  * Reintroduces runtime dependency

### 3. Framework-Specific Output (e.g., Next.js `.next/`)

* **Description**: Preserve framework build artifacts
* **Rejected because**:

  * Locks architecture to a specific toolchain
  * Unnecessary complexity

---

## Consequences

### Positive

* Extremely simple hosting and caching
* Excellent compatibility with search engines and AI crawlers
* Easy inspection and debugging
* Clear publish/unpublish semantics

### Negative

* Directory count may grow large at scale
* Requires careful slug uniqueness enforcement

---

## Impacted Areas

* Publish pipeline
* Search indexing (AutoRAG)
* Sitemap generation
* Pages routing configuration

---

## Compatibility With Architecture Principles

This decision reinforces:

* Static-first publishing
* File-native content storage
* Infrastructure-level simplicity

as defined in `ARCHITECTURE.md`.

---

## Reconsideration Triggers

This ADR may be revisited if:

* R2 object listing performance becomes a bottleneck
* URL structure requirements fundamentally change

Any change must be documented via a superseding ADR.

---

## Related Documents

* ADR-0001: Static Publish for User-Facing Pages
* ADR-0002: Why ZeroPress Is Not a Headless CMS
* ARCHITECTURE.md

---

## Notes

This ADR defines the physical persistence contract of ZeroPress content.
It should be treated as a backward-compatibility boundary.
