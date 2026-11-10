# Property Development Intelligence (PDI)

Documentation-only product scope for an **AI-native Property Development Intelligence and Feasibility Platform** (Kenya-first). This repository holds architecture, pipelines, and funding strategy for a greenfield product. Implementation patterns align with the [EpiCare](https://github.com/bkb-Git/epi-care) stack (Next.js, Convex, Mastra, Clerk) but PDI is a separate product.

**Working title:** Property Development Intelligence (PDI).

## Start here

| Document | Purpose |
|----------|---------|
| [PRODUCT_SCOPE.md](./PRODUCT_SCOPE.md) | Vision, users, moat, releases, competitive landscape, decision log |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | Engines vs modules vs agents vs configurable packs |
| [PIPELINES.md](./PIPELINES.md) | Pipeline A (design) vs Pipeline B (viability) |
| [DESIGN_PROPOSAL_CONTRACT.md](./DESIGN_PROPOSAL_CONTRACT.md) | Semantic contract between design and financial engines |
| [ONTOLOGY.md](./ONTOLOGY.md) | Shared property domain model |
| [DATA_SOURCES.md](./DATA_SOURCES.md) | Kenya data sources and agent data matrix |
| [OPEN_SOURCE_DESIGN_STACK.md](./OPEN_SOURCE_DESIGN_STACK.md) | OSS vs paid design tools |
| [FUNDING_PIPELINE.md](./FUNDING_PIPELINE.md) | Grants, VC, and application positioning |
| [TECH_STACK.md](./TECH_STACK.md) | EpiCare parity and net-new dependencies |

## Core architectural principle

Do **not** build one large AI agent with tools. Build a **property intelligence system** with a shared domain model, deterministic calculation engines, specialist agents, and an evidence-backed decision layer. AI interprets, investigates, and challenges; the system owns facts, calculations, constraints, and provenance.

## Problem framing

A property development is a constrained optimisation problem:

\[
\max_{d \in D} \; \text{RiskAdjustedValue}(d)
\]

Subject to legal, physical, capital, timeline, and market demand constraints—where \(d\) is a development configuration. The platform must recommend **better** configurations than the developer’s initial proposal when economics and constraints support it.
