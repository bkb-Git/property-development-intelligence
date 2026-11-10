# Property Development Intelligence (PDI)

**AI-native Property Development Intelligence and Feasibility Platform** (Kenya-first). This repository holds product scope, architecture, and funding specifications for a greenfield build. Implementation patterns align with the [EpiCare](https://github.com/bkb-Git/epi-care) stack (Next.js, Convex, Mastra, Clerk); PDI is a separate product.

**Working title:** Property Development Intelligence (PDI).

## Documentation

All specifications live under [`docs/`](./docs/README.md).

| Area | Entry |
|------|--------|
| **Start here** | [docs/product/PRODUCT_SCOPE.md](./docs/product/PRODUCT_SCOPE.md) |
| **Full index** | [docs/README.md](./docs/README.md) |

## Core principle

Do **not** build one large AI agent with tools. Build a **property intelligence system** with a shared domain model, deterministic calculation engines, specialist agents, and an evidence-backed decision layer.

## Problem framing

\[
\max_{d \in D} \; \text{RiskAdjustedValue}(d)
\]

subject to legal, physical, capital, timeline, and market constraints—where \(d\) is a development configuration.
