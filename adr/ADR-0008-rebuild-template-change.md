ADR-0008: Rebuild & Template Change Policy
==========================================

*   **Status**: Accepted
*   **Date**: 2025-12-23
*   **Deciders**: ZeroPress Core Team
*   **Related ADRs**:
    *   ADR-0005: HTML Minimal Generation Strategy
    *   ADR-0007: Source of Truth (D1) and Artifact Store (R2)

* * *

Context
-------

ZeroPress publishes content by generating static HTML artifacts from source content stored in D1 and templates stored in source control.  
Published HTML files are stored in Cloudflare R2 and served via Pages/CDN.

As the system evolves, template changes (layout, components, metadata structure, accessibility, SEO improvements) are expected to occur frequently.  
A clear policy is required to define:

*   How template changes affect existing published posts
*   Whether partial rebuilds are supported
*   How rebuilds relate to fault tolerance and recovery

* * *

Decision
--------

ZeroPress adopts a **full rebuild / re-publish policy** for template changes.

### Core rules

1.  **Templates are not version-pinned per post**
2.  **Any template change triggers re-publication of all published content**
3.  **Published HTML is always considered a reproducible artifact**
4.  **R2 is overwritten, not migrated**

Template changes are treated as **build system changes**, not data migrations.

* * *

Rationale
---------

### 1\. Deterministic publishing

Published output is defined as:

```
HTML = content (D1) + template + build logic
```

When templates change, the deterministic output must be regenerated to preserve consistency.

* * *

### 2\. Operational simplicity

*   No per-post template version tracking
*   No backward compatibility layer
*   No mixed-template states across the site

This keeps the publishing model understandable and predictable.

* * *

### 3\. Fault tolerance alignment

ZeroPress treats R2 as an **artifact store**, not a backup target.

*   Losing R2 → re-publish
*   Changing templates → re-publish

Both cases follow the same recovery path.

* * *

### 4\. SEO and UX consistency

A full rebuild ensures:

*   Consistent metadata structure
*   Uniform accessibility improvements
*   Predictable layout and navigation
*   No fragmented user experience across old/new posts

* * *

Consequences
------------

### Positive

*   Clear mental model for users and contributors
*   Simplified recovery and rebuild processes
*   Template improvements automatically propagate to all content
*   Strong alignment with static-site and build-based publishing principles

### Negative

*   Template changes require rebuilding all published posts
*   Large sites may incur higher build time and R2 write volume

These costs are considered acceptable given modern infrastructure capabilities and the benefits of consistency.

* * *

Non-Goals
---------

This ADR explicitly does **not** support:

*   Per-post template version locking
*   Partial or incremental template-only rebuilds
*   Runtime template rendering from R2 or D1
*   Database-level HTML storage as a primary delivery mechanism

Such features may be reconsidered only if ZeroPress evolves into a large-scale media platform with regulatory or archival requirements.

* * *

Future Considerations
---------------------

The following may be explored in future ADRs if needed:

*   Template version pinning for legal or archival use cases
*   Differential rebuild strategies
*   Template compatibility checks before publish
*   Rebuild scheduling and rate limiting

None of these are required for the current ZeroPress scope.

* * *

Summary
-------

> In ZeroPress, a template change is a rebuild event.

Published HTML is disposable, reproducible, and replaceable.  
Consistency, simplicity, and recoverability take precedence over incremental optimization.
