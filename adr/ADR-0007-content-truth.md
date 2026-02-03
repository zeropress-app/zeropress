# ADR-0007: Content Truth, Drafts, and Revisions

## Status

Accepted

## Context

ZeroPress is designed as a document-first publishing system. Prior architectural decisions establish that:

* User-facing pages are delivered as complete HTML documents (ADR-0001)
* ZeroPress is not a Headless CMS and does not render content at runtime (ADR-0002)
* R2 file layout is a stable, long-lived contract (ADR-0003)
* Search and AI indexing are secondary, non-authoritative systems (ADR-0004)
* Client-side hydration is strictly limited to decoration (ADR-0005)
* Client-side enhancements must never re-render content (ADR-0006)

As ZeroPress introduces drafts, revisions, previews, and editorial workflows, it becomes necessary to clearly define where content may exist, which representations are authoritative, and how transitions between editorial state and published documents occur.

Without a strict definition of content truth, the system risks drifting toward a database-centric CMS model where HTML becomes a derived cache rather than the primary artifact.

## Decision

ZeroPress formally distinguishes between **editorial content state** and **published documents**, and assigns strict authority boundaries between D1 and R2.

### Core Rule

> **Only published HTML documents stored in R2 are authoritative user-facing content.**

All other representations of content are non-authoritative editorial states.

## Storage Responsibilities

### R2 (Document Store — Authoritative)

R2 stores:

* Canonical, published HTML documents
* Files directly served to users
* Documents addressed by stable public URLs

Characteristics:

* Immutable by default
* Cacheable at CDN and browser level
* Required for site availability

If a document does not exist in R2, it is considered unpublished.

### D1 (Editorial State Store — Non-authoritative)

D1 stores:

* Draft content (Markdown, HTML fragments, JSON, editor state)
* Revision history
* Publishing metadata (status, timestamps, author)
* Slugs, routing metadata, and R2 object references
* Workflow and permission state

Characteristics:

* Mutable
* Admin-only
* Not directly exposed to user pages

Content stored in D1 is not considered published, regardless of completeness or similarity to the final HTML.

## Drafts and Revisions

Drafts and revisions are defined as **editorial candidates**, not documents.

Rules:

* Drafts and revisions exist only in D1
* They have no canonical public URL
* They must not be indexed, crawled, or served to users
* They may structurally resemble final content, but carry no authority

Revisions may be used for:

* Preview rendering
* Diffing and rollback
* Re-publishing workflows

A revision becomes authoritative only after it is rendered and written to R2 as the canonical document.

## Publish Semantics

Publishing is a semantic boundary-crossing event:

1. Editorial state is read from D1
2. HTML is rendered
3. The rendered document is written to R2
4. The R2 write defines publication success
5. Metadata is recorded or updated in D1

Failure semantics:

* If R2 write fails, publish fails
* If D1 metadata update fails, publish may be retried

This establishes R2 as the system of record for published content.

## Canonical vs Non-canonical Documents

ZeroPress supports a single canonical document per public URL.

Optional non-canonical artifacts (e.g., internal previews or revision renders) may exist under restricted paths, provided that:

* They are not reachable via user routing
* They are excluded from sitemaps and search indexing
* They are never treated as authoritative

## Failure Scenarios

### D1 Unavailable

* User pages continue to function
* Publishing and editing are temporarily unavailable
* This is considered an acceptable and expected degradation

### R2 Unavailable

* User pages are unavailable
* Publishing fails
* This is considered a critical failure

These failure modes reinforce R2 as the authoritative store.

## Consequences

### Benefits

* Clear separation between editorial workflow and published content
* Strong guarantees around content durability and caching
* Elimination of runtime rendering dependencies
* Predictable SEO and AI ingest behavior

### Trade-offs

* Draft content loss is possible if D1 is compromised
* Recovery from R2 loss requires re-publishing, not restoration
* Editorial convenience is intentionally subordinated to document integrity

## Summary

ZeroPress treats content as a **published document**, not as a database record.

Drafts and revisions may exist in D1 as editorial state, but only HTML written to R2 defines reality.

> **If it is not in R2, it is not published.**
