---
slug: "catdocs"
title: "Catdocs: Managing Modular OpenAPI Specifications at Scale"
author: imun
post_type: blog
description: "CLI tool and .NET library that addresses this by allowing OpenAPI files to be split into modular components, validated, and reassembled reliably."
summary: "By using Catdocs, each team can manage their part of the API independently—paths, schemas, components—while still contributing to a single source of truth that is validated and bundled in CI."
category: "projects"
tags:
    - openapi
    - dotnet
    - csharp
    - automation
    - ci-cd
cover_image: "https://github.com/imaun/catdocs/blob/master/assets/catdocs-header.png"
created_at: "2025-06-10 20:00:00"
updated_at: "2025-06-10 20:00:01"
---
Maintaining a large OpenAPI specification as a single file often leads to complexity, especially in team environments. Merge conflicts, misplaced references, and bloated definitions make collaboration error-prone and slow.

**[Catdocs](https://github.com/imaun/catdocs)** is a CLI tool and .NET library that addresses this by allowing OpenAPI files to be split into modular components, validated, and reassembled reliably.

## A Practical Use Case: Coordinating API Work Across Teams
Consider a fintech company with a REST API that spans multiple services: authentication, payments, account management, etc. Each service is handled by a separate team. Maintaining a unified OpenAPI spec in this setup introduces friction:

- Developers are constantly overwriting each other’s changes.
- Pull requests frequently break due to unresolved `$ref` paths.
- Versioning the API becomes tedious without a structure.

By using **Catdocs**, each team can manage their part of the API independently—paths, schemas, components, while still contributing to a single source of truth that is validated and bundled in CI pipelines.

## Overview: What Catdocs Provides
Catdocs is designed for:
- Modular OpenAPI development.
- CLI-based validation and processing.
- Integration into build systems and CI/CD workflows.
- Teams who work on different parts of a shared API.

### Core Capabilities:
- `split`: Break a large spec into reusable files.
- `bundle`: Combine modular files into a valid OpenAPI document.
- `convert`: Transform specs between YAML and JSON.
- `stats`: Analyze and validate OpenAPI documents.

It supports both OpenAPI 2 and 3, and works with either `.yaml` or `.json`.

---

## Getting Started

### Install via Git
```bash
git clone https://github.com/imaun/catdocs.git
cd catdocs
dotnet build
```

### Run a command (example: get stats)
```bash
dotnet run -- stats --file path/to/your/openapi.yaml --format yaml
```

### Or use Docker
```bash
docker pull imaun/catdocs:latest
docker run --rm imaun/catdocs:latest stats --file path/to/your/openapi.yaml
```

---

## CLI Commands Overview

### 0. `stats`: Analyze and validate
Validates the spec and prints structural stats (paths, schemas, responses).

```bash
catdocs stats --file example-openapi.yaml --spec-ver 3 --format json
```

---

### 1. `split`: Break down the spec
Splits a unified OpenAPI document into smaller, manageable files.

```bash
catdocs split --file OpenApi.yaml --spec-ver 3 --format yaml --outputDir examples/bundle-pipeline
```

After running, the directory will contain separate folders for `paths`, `components`, etc.

---

### 2. `bundle`: Recombine split files
Combines the split files into a single OpenAPI document, resolving references.

```bash
catdocs bundle --file examples/bundle-pipeline/OpenApi.yaml \
  --spec-ver 3 --format yaml --output examples/bundle-pipeline/output.yaml
```

---

### 3. `convert`: JSON ⇄ YAML
Converts OpenAPI spec format while validating structure.

```bash
catdocs convert --file docs.json --spec-ver 3 --format json --output docs.yaml
```

---

## CI/CD Integration Example

In a CI pipeline, use `bundle` to ensure all components merge correctly after each commit:

```yaml
# .github/workflows/openapi.yaml
name: Validate and Bundle OpenAPI

on:
  push:
    branches: [main]

jobs:
  bundle:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Catdocs
        run: |
          docker run --rm -v ${{ github.workspace }}:/data \
            imaun/catdocs:latest bundle \
            --file /data/examples/bundle-pipeline/OpenApi.yaml \
            --output /data/final-openapi.yaml
```

---

## .NET Integration

Catdocs internally uses the official `Microsoft.OpenApi` package for parsing and processing OpenAPI specifications. This ensures compatibility with widely accepted OpenAPI standards and allows direct reuse in .NET projects.

You can use Catdocs as a library in your .NET apps to programmatically manipulate OpenAPI documents, leveraging the same `Microsoft.OpenApi` internals.

```bash
dotnet add package Catdocs.OpenAPI
```

Example usage:
```csharp
using Catdocs.OpenAPI;
// Access and manipulate OpenAPI specs directly
```
---

## When Not to Use Catdocs
While Catdocs solves modular API management well, it's not universally beneficial:

- If your API spec is small and managed by one person, splitting may overcomplicate things.
- It does not support visual previews like Redocly Studio.
- Teams unfamiliar with OpenAPI structure may introduce incorrect `$ref` paths.

---

## Summary
Catdocs offers a set of tooling that makes modular OpenAPI workflows more reliable. It doesn’t replace full-blown API platforms, but provides just enough structure and automation for teams that want flexibility without vendor lock-in.

Repository: **[github.com/imaun/catdocs](https://github.com/imaun/catdocs)**

---
[🔗 Source for this blog post](https://github.com/imaun/website/blob/master/data/blog/posts/catdocs.md)