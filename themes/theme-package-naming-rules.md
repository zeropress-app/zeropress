# ZeroPress Theme Package Naming Rules

This document defines the canonical naming rules for ZeroPress theme packages.

It is a public contract for:

- package identity
- namespace registration
- theme slug rules
- version formatting
- download URL structure
- internal storage path conventions

The goal is to keep theme package names stable, collision-resistant, and easy to reason about across the ZeroPress ecosystem.

## Canonical Package Identity

Each theme package is identified by:

```text
{namespace}.{slug}
```

Examples:

```text
official.blog
acme.docs-theme
lael.minimal-starter
```

Rules:

- `namespace` identifies the publisher
- `slug` identifies the theme within that publisher namespace
- the pair `{namespace}.{slug}` is the canonical package identity
- theme slugs may repeat across different namespaces
- the same `{namespace}.{slug}` pair must never be reused for a different theme

## Canonical Download URL

Public downloads use the following format:

```text
/theme/{namespace}.{slug}@{version}.zip
```

Examples:

```text
/theme/official.blog@1.2.0.zip
/theme/acme.docs-theme@1.0.0.zip
/theme/lael.minimal-starter@0.0.2.zip
```

This format is the public contract. Clients should treat it as stable.

## Package Filename Structure

The package filename is:

```text
{namespace}.{slug}@{version}.zip
```

Components:

| Component | Meaning |
| --- | --- |
| `namespace` | Publisher or organization identifier |
| `slug` | Theme identifier within the namespace |
| `version` | Theme version in SemVer format |

## Namespace Rules

The namespace identifies the publisher of the theme.

Examples:

```text
official
acme
my-company
team42
lael
```

### Allowed Characters

- lowercase letters: `a-z`
- digits: `0-9`
- hyphen: `-`

### Pattern

```regex
^[a-z0-9]+(-[a-z0-9]+)*$
```

### Rules

- lowercase only
- may contain hyphens
- hyphens must appear only between alphanumeric characters
- must not start with `-`
- must not end with `-`
- consecutive hyphens are not allowed

Valid examples:

```text
acme
acme-inc
my-company
team42
```

Invalid examples:

```text
-acme
acme-
acme--inc
Acme
acme_inc
acme.inc
acme inc
```

### Length

Required:

```text
3-24 characters
```

### Uniqueness

Namespaces are globally unique.

Valid:

```text
official
acme
lael
```

Invalid:

```text
official
official
```

### Immutability

Namespaces must be immutable once created.

Reasons:

- package identity stability
- download URL permanence
- storage path stability
- long-term ecosystem consistency

### Reserved Namespaces

The following namespaces must not be registered by regular users:

```text
default
official
system
core
admin
plugin
theme
themes
api
www
example
test
```

Notes:

- reserved namespaces may be used only by the ZeroPress platform if needed
- examples in this document do not use `default` as a user-registrable namespace

### Official Package Examples

Examples of platform-owned package identities:

```text
official.blog
official.docs
official.minimal
```

## Slug Rules

The slug identifies the theme within a namespace.

Examples:

```text
minimal
minimal-starter
clean-blog
docs-theme
portfolio
```

### Allowed Characters

- lowercase letters: `a-z`
- digits: `0-9`
- hyphen: `-`

### Pattern

```regex
^[a-z0-9]+(-[a-z0-9]+)*$
```

### Rules

- lowercase only
- may contain hyphens
- hyphens must appear only between alphanumeric characters
- must not start with `-`
- must not end with `-`
- consecutive hyphens are not allowed

Valid examples:

```text
blog
clean-blog
minimal-starter
docs-theme
portfolio
```

Invalid examples:

```text
-blog
blog-
clean--blog
blog------
Blog
clean_blog
clean.blog
clean blog
```

### Length

Required:

```text
3-32 characters
```

### Uniqueness

Slugs must be unique within a namespace, but may be reused across different namespaces.

Valid:

```text
official.blog
acme.blog
lael.blog
```

Invalid:

```text
official.blog
official.blog
```

### Immutability

Slug changes are strongly discouraged after publication.

If a slug changes, all of the following may need to change:

- package identity
- download URL
- internal storage path
- references from external systems

For that reason, a published slug should be treated as effectively immutable.

## Version Rules

Versions follow Semantic Versioning.

Format:

```text
major.minor.patch
```

Examples:

```text
1.0.0
1.2.3
2.0.0
0.1.0
```

### Pattern

```regex
^[0-9]+\.[0-9]+\.[0-9]+$
```

### Notes

- pre-release versions are not part of the current public download contract
- versions such as `1.0.0-beta` or `1.2.0-rc1` may be supported in the future, but should not be used in the current public package URL format

## Parsing Rules

The package filename can be parsed using simple delimiters:

- split once at `@` to separate identity from version
- remove the trailing `.zip`
- split the identity once at the first `.` to separate namespace from slug

Example:

```text
official.minimal-starter@1.0.0.zip
```

Parses as:

```text
namespace = official
slug = minimal-starter
version = 1.0.0
```

Important:

- `namespace` must not contain `.`
- `slug` must not contain `.`
- `version` must not contain `@`
- clients should validate each parsed component against the rules in this document

## Internal Storage Path

The public download URL is:

```text
/theme/{namespace}.{slug}@{version}.zip
```

Internal object storage may map this to:

```text
/themes/{namespace}/{slug}/{version}.zip
```

Example:

```text
/themes/official/minimal-starter/1.0.0.zip
```

This internal path is an implementation detail, but it is the recommended structure because it:

- groups artifacts by publisher
- groups versions by theme
- avoids a single flat artifact directory

## Summary

| Component | Pattern | Notes |
| --- | --- | --- |
| `namespace` | `^[a-z0-9]+(-[a-z0-9]+)*$`, length `3-24` | globally unique, immutable, reserved words blocked |
| `slug` | `^[a-z0-9]+(-[a-z0-9]+)*$`, length `3-32` | unique within namespace, effectively immutable after publish |
| `version` | `^[0-9]+\.[0-9]+\.[0-9]+$` | current public contract uses stable SemVer only |

Canonical package identity:

```text
{namespace}.{slug}
```

Canonical download filename:

```text
{namespace}.{slug}@{version}.zip
```

Canonical public download URL:

```text
/theme/{namespace}.{slug}@{version}.zip
```

Recommended internal storage path:

```text
/themes/{namespace}/{slug}/{version}.zip
```
