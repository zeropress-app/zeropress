# ADR: ZeroPress Theme System Design

## Status

Accepted (Proposed for implementation)

## Context

ZeroPress is a cloud-native CMS built on Cloudflare infrastructure (Pages, Workers, D1, KV, R2). Its primary positioning is as a **WordPress alternative** that removes operational burden (server management, upgrades, security patches) while remaining flexible for both developers and non-developers.

A critical architectural decision is how to design the **theme system**, as this directly impacts:

* Adoption by existing WordPress users
* Usability for non-developers
* Performance and cacheability on the edge
* Security and long-term maintainability

Historically, popular static site generators (e.g., Jekyll, early Astro use cases) relied on Markdown-based static builds. While technically capable, these systems primarily targeted developers and made content editing, menu changes, layout adjustments, and theme customization cumbersome for non-technical users.

WordPress themes, on the other hand, are powerful but introduce significant complexity:

* PHP runtime dependency
* Global state and implicit execution flow
* Plugin-driven side effects
* Difficult caching and unpredictable performance

ZeroPress must avoid repeating these issues while still offering a familiar and approachable theming experience.

---

## ZeroPress 기준 대안 비교

### Option A: WordPress-like Theme System (PHP-style, logic-heavy)

**Description**

* Themes behave like applications
* Arbitrary logic, hooks, filters
* Strong resemblance to WordPress for familiarity

**Pros**

* Low initial friction for WordPress developers
* Existing mental model

**Cons**

* Violates Cloudflare edge execution principles
* Poor cacheability
* High security risk
* Non-developer unfriendly
* Recreates WordPress maintenance problems

**Assessment**
Rejected. This contradicts ZeroPress’s core value: *"The CMS that runs itself."*

---

### Option B: Static Site Generator-style Themes (Build-time only)

**Description**

* Markdown-first
* Build step required
* Developer-centric workflow

**Pros**

* High performance
* Predictable output

**Cons**

* Editing content requires rebuilds
* Menu/layout/theme changes are not intuitive
* Not suitable for non-developers

**Assessment**
Rejected. This limits ZeroPress to a developer-only audience and conflicts with CMS expectations.

---

### Option C: ZeroPress Declarative Theme System (Selected)

**Description**
Themes are **not applications**, but **declarative layouts** composed of:

* HTML templates
* CSS assets
* Explicit slot placeholders
* Read-only data contracts

Themes contain **no executable business logic** and do not control data fetching or routing.

**Core Principle**

> Theme ≠ App
> Theme = Layout + Slots + Data Contract

**Pros**

* Fully compatible with edge caching
* High security by design
* Predictable rendering
* Easy to customize for non-developers
* Minimal cognitive load

**Cons**

* Less expressive than full programming models
* Requires clear documentation of data contracts

**Assessment**
Accepted. This model aligns with ZeroPress’s goals and infrastructure constraints.

---

## Decision

ZeroPress will implement a **declarative, HTML-based theme system** with the following constraints:

1. Themes must not execute arbitrary logic
2. Themes may only consume predefined data contracts
3. Rendering is handled by ZeroPress Workers
4. Themes are safe to cache aggressively at the edge

---

## Theme Structure (Proposed)

```
theme/
├ layout.html
├ post.html
├ page.html
├ partials/
│  ├ header.html
│  └ footer.html
├ assets/
│  ├ style.css
│  └ theme.js (optional, restricted)
└ theme.json
```

---

## Rendering Model

1. Request arrives at Cloudflare Worker
2. Worker resolves content and data
3. HTML templates are loaded
4. Slots are replaced with content fragments
5. Final HTML is returned and cached

No theme-level control flow is permitted.

---

## Consequences

### Positive

* Strong differentiation from WordPress
* Safer ecosystem
* Clear mental model for users
* Low operational overhead

### Trade-offs

* Advanced conditional logic must be handled at the CMS level
* Theme authors must adapt to a declarative mindset

---

## Migration & Positioning Implications

Messaging to WordPress users:

* “Themes won’t break your site.”
* “Customizable like WordPress, predictable like static.”
* “Edit HTML and CSS, not server code.”

This decision reinforces ZeroPress as a **modern CMS**, not merely a static site generator or a WordPress clone.

---

## References

* WordPress Theme Architecture
* Cloudflare Workers execution model
* Static site generator limitations in CMS use cases
