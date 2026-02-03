# ADR-0006: Content Enhancement Policy (Decoration vs Rendering)

## Status

Accepted

## Context

ZeroPress is a file-centric publishing system where HTML is the primary delivery artifact and the canonical representation of published content. Previous ADRs establish strict constraints:

* Content is published as complete HTML documents (ADR-0001)
* ZeroPress is not a Headless CMS and does not rely on runtime APIs to render content (ADR-0002)
* R2 file layout is a persistence contract (ADR-0003)
* Search is a secondary, non-authoritative system (ADR-0004)
* Client-side hydration is strictly limited (ADR-0005)

As the platform matures, there is a need to clarify which client-side enhancements are acceptable without violating ZeroPress’s document-first philosophy. In particular, features such as syntax highlighting, image lightboxes, copy buttons, and minor UX enhancements may require DOM manipulation.

Without a clear boundary, such enhancements risk evolving into client-side rendering, rehydration, or SPA-like behavior.

## Decision

ZeroPress formally distinguishes between **Content Rendering** and **Content Enhancement**.

### Definitions

**Content Rendering**

* The process of interpreting structured data (JSON, Markdown, DB records) into HTML
* Occurs only at publish time
* Produces the final HTML document stored in R2

**Content Enhancement**

* Optional, client-side DOM mutations that decorate or augment existing HTML
* Must not reinterpret, regenerate, or replace content
* Must preserve the original semantic text and document structure

Only Content Enhancement is permitted at runtime.

## Allowed Enhancements

The following are explicitly allowed:

* Syntax highlighting that wraps existing text nodes with `<span>` elements
* Image lightbox modals triggered from existing `<img>` elements
* Copy-to-clipboard buttons for code blocks
* Anchor link generation for headings
* Table-of-contents highlighting based on scroll position

Constraints:

* Enhancements must operate on existing DOM nodes
* Enhancements must not fetch content data to re-render sections
* Enhancements must not require framework hydration (React, Vue, etc.)

## Disallowed Behavior

The following are explicitly forbidden on user pages:

* Re-rendering content from JSON or Markdown at runtime
* Client-side routing that intercepts normal hyperlink navigation
* React/Vue/Svelte hydration of content areas
* Replacing `<main>` or article bodies via JavaScript
* Treating the page as an application shell

Any behavior that causes content to be reconstructed rather than decorated is considered a policy violation.

## SEO and AI Considerations

* Tokenized HTML (e.g., syntax-highlighted code blocks using `<span>` wrappers) is acceptable
* Search engines and AI systems treat such markup as semantic noise and extract plain text
* `<pre><code>` structures must be preserved to maintain machine readability

Publish-time tokenization is preferred over runtime processing whenever possible.

## Consequences

### Positive

* Ensures user pages remain durable documents
* Prevents gradual drift toward SPA architectures
* Maintains excellent SEO and AI ingestibility
* Keeps performance predictable and cache-friendly

### Trade-offs

* Limits the use of modern JS frameworks on user pages
* Requires stricter review of client-side enhancements
* Some interactive patterns must be implemented via Workers or separate endpoints

## Summary

This ADR acts as a constitutional boundary for ZeroPress.

Client-side JavaScript may decorate documents, but it must never render them.

Once published, a ZeroPress page is a document — not an application.
