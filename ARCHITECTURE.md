# ZeroPress Architecture

This document records the **core architectural decisions** of ZeroPress.
It focuses on decisions that are intentionally hard to change and serves as a long-term reference for maintainers and contributors.

---

## Purpose

ZeroPress is designed as a modern, file-based, AI-search–enabled CMS built on Cloudflare infrastructure.
This document explains **why ZeroPress is designed the way it is**, not how to use it.

---

## Non-Goals

ZeroPress explicitly does **not** aim to:

* Be a full SPA framework
* Render user pages dynamically at request time
* Depend on a traditional database for user-facing pages
* Replicate the WordPress plugin or theme ecosystem
* Provide per-request personalized content rendering

---

## Core Principles

1. **User pages must be fully static** and readable without JavaScript
2. **Published content must exist as files**
3. **Failure should degrade to static HTML**, not to an error page
4. **Admin and User pages are strictly separated**
5. **Search must not affect page availability**
6. **Infrastructure complexity must stay invisible to users**

---

## High-Level Architecture

ZeroPress is composed of four main layers:

* **Cloudflare Pages**: Serves all user-facing pages
* **Cloudflare R2**: Stores published content as files
* **Cloudflare Workers**: Handles dynamic features
* **Cloudflare AutoRAG**: Provides AI-powered search

```
Admin → Worker → R2 → Pages → User
                ↘ AutoRAG
```

---

## Content Model

All published content exists as files in R2.

Canonical structure:

```
/public/posts/{slug}/index.html   # Canonical HTML
/public/posts/{slug}/meta.json    # Auxiliary structured data
```

* `index.html` acts as a deterministic renderer of content derived from the source of truth.
* `meta.json` is used for:

  * AI search (AutoRAG)
  * Client-side hydration
  * Optional fallback search

Draft or private content must never be stored under `/public`.

---

## Rendering Strategy

* HTML is generated **at publish time**, not at request time
* No server-side rendering on user requests
* A single HTML template may be reused for thousands of pages
* JavaScript hydration is **optional and non-critical**

If JavaScript fails, the page must remain readable.

---

## Navigation Strategy

* Direct access and external entry always load static HTML
* Internal navigation may use soft navigation (PJAX-style)
* Full SPA routing is intentionally avoided

---

## Search Architecture

* Search result pages are **not statically generated**
* Search UI pages are marked as `noindex`
* Search is powered by **Cloudflare AutoRAG**, crawling R2 content
* Search failures must not impact page availability

Search is considered a **tool**, not a content surface.

---

## Sitemap Strategy

* Sitemaps are generated from actual files stored in R2
* Only existing `index.html` files are included
* Search result pages are excluded

---

## Trade-offs

ZeroPress intentionally accepts the following trade-offs:

* Delayed search indexing due to periodic crawling
* Limited real-time personalization
* No dynamic per-request rendering

These trade-offs are accepted in favor of:

* Performance
* Reliability
* Security
* Operational simplicity

---

## Reconsideration Triggers

These architectural decisions may be revisited if:

* Static HTML becomes a scalability bottleneck
* Cloudflare AutoRAG is no longer viable
* A strong, unavoidable use-case requires request-time rendering

Any such change must be explicitly documented.

---

## Guiding Philosophy

> ZeroPress is built on the belief that **content should exist independently of applications**,
> and that **AI should understand content as it is published**, not as it is queried.
