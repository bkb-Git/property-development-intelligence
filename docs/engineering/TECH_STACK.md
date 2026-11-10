# Technology stack

PDI targets **EpiCare-parity** patterns for product engineering, with additive **Python worker** and **geospatial/BIM** services. This document describes the **target** stack when the product is scaffolded—not current EpiCare dependencies.

---

## EpiCare patterns to reuse

| Layer | EpiCare | PDI application |
|-------|---------|-----------------|
| Monorepo | Bun workspaces, Turbo, Vitest | Same |
| Dashboard | Next.js 14, shadcn, Clerk | New app `apps/pdi-dashboard` (future) or standalone repo |
| Realtime / jobs | Convex tables, crons, actions | `designJobs`, `viabilityJobs`, projects, evidence |
| RAG | `@convex-dev/rag` | Regulatory document retrieval |
| AI | Mastra, `runSequentialTools`, agent webhooks | Specialist agents only; no sync multi-LLM in API routes |
| Async | `dispatchAgentWorkflow`, `waitUntil`, UI polling | Design + viability pipelines |
| Documents | `@epicare/document-extract` | Title deeds, surveys, BOQs, email/PDF |
| Export | `@epicare/document-export` patterns | IC report PDF |
| Auth | Clerk | Org/tenant boundary |
| Client HTTP data | RTK Query where REST to Python worker | Engine invocation from dashboard |

Reference: [EpiCare `mastra-async-ops`](https://github.com/bkb-Git/epi-care/blob/main/.cursor/rules/mastra-async-ops.mdc), [EpiCare intelligence layer guide](https://github.com/bkb-Git/epi-care/blob/main/docs/plans/INTELLIGENCE_LAYER.md).

---

## Convex vs PostgreSQL / PostGIS

| Concern | v1 (R1–R2) | R3+ |
|---------|------------|-----|
| Projects, jobs, evidence, rules metadata | **Convex** | Convex |
| Vector search (regulations) | Convex RAG | Convex RAG |
| Heavy spatial joins, county overlays | GeoJSON in object storage + Turf client | **PostGIS adjunct** (Supabase or dedicated Postgres) synced by `parcelId` |
| Financial/spatial compute | **Python worker** | Same |

Do not duplicate parcel truth: one canonical `parcelId`; PostGIS is a spatial index, not a second project store.

---

## Repository layout (future)

```text
apps/
  pdi-dashboard/          # Next.js (clone patterns from apps/dashboard)
packages/
  pdi-contracts/          # Zod: DesignProposal, ViabilityAssessment
  pdi-config/             # County packs, ontology JSON (versioned)
  pdi-financial/          # Optional TS wrappers; core math in Python
services/
  pdi-engine/             # FastAPI: fin, cost, geo, ifc, optimisation
convex/
  pdi/                    # If co-deployed with EpiCare; or separate Convex project
```

Docs-only phase: specifications live in this repository (implementation not yet scaffolded).

---

## TypeScript / dashboard packages (net-new)

| Package | Purpose |
|---------|---------|
| `maplibre-gl` | Parcel and site maps |
| `@maplibre/maplibre-gl-geocoder` | Geocoding (provider TBD) |
| `@turf/turf` | Client buffers, area sanity checks |
| `@types/geojson`, `wellknown` | WKT/GeoJSON interchange |
| `zod` | Shared with `pdi-contracts` |

Existing EpiCare packages to reuse when co-located:

- `@epicare/shared` — patterns for config/feature flags  
- `@epicare/document-extract` — PDF/DOCX ingestion  
- `@epicare/document-export` — report PDF  

---

## Python `pdi-engine` service (net-new)

| Dependency | Role |
|------------|------|
| `fastapi`, `uvicorn` | HTTP API for workers |
| `numpy`, `pandas` | Financial tables, sensitivity |
| `geopandas`, `shapely` | Parcel operations |
| `ortools`, `pymoo` | Layout / mix optimisation |
| `ifcopenshell` | Quantity takeoff from IFC |

Optional:

- `redis`, `rq` or Celery — job queue if worker load exceeds webhook fan-out  
- `pydantic` — mirror Zod contracts  

**Invocation pattern (EpiCare-aligned):**

1. Dashboard webhook receives job.  
2. Mastra orchestrates LLM steps.  
3. Engine calls `POST /pdi-engine/v1/...` with signed internal auth.  
4. Worker returns ledger JSON + file URLs.  
5. Convex callback stores results (no OpenRouter inside Convex except embeddings if shared).  

---

## Headless OSS (not npm — worker OS packages)

| Tool | Role |
|------|------|
| Blender + Sverchok add-on | Procedural massing |
| QGIS CLI | Batch overlay processing |
| Bonsai (Blender BIM) | IFC authoring |
| Ladybug Tools | Environmental (R3+) |

Container images version-pin Blender/QGIS for reproducibility.

---

## Optional paid integrations (MCP / API)

| Tool | Role |
|------|------|
| TestFit | Site planning export → contract normalisation |
| Autodesk Forma / APIs | Environmental massing |
| Revit | Professional export target |

Default product path remains OSS; integrations behind feature flags.

---

## Mapping layers to implementation

| Layer | Technology |
|-------|------------|
| GIS and site intelligence | QGIS, PostGIS, GeoPandas, MapLibre |
| Design geometry | Blender, Sverchok, Python |
| Spatial relationships | TopologicPy |
| BIM and IFC | Bonsai, IfcOpenShell |
| Design optimisation | OR-Tools, pymoo |
| Environmental analysis | Ladybug Tools, EnergyPlus (later) |
| Agent orchestration | Mastra (LangGraph acceptable alternative; pick one) |
| Financial calculations | Python `pdi-engine` (source of truth) |
| Shared data and evidence | Convex + object storage |
| Auth | Clerk |

---

## Feature flags (release gates)

Mirror EpiCare `globalConfig.featureFlags` style:

| Flag | Release |
|------|---------|
| `pdi_feasibility_core` | R1 |
| `pdi_market_comps` | R2 |
| `pdi_regulatory_rag` | R2 |
| `pdi_comms_ledger` | R2 |
| `pdi_design_pipeline` | R2–R3 |
| `pdi_postgis` | R3 |
| `pdi_mix_optimisation` | R4 |
| `pdi_testfit_integration` | Optional tier |

---

## Security and compliance notes

- No patient PHI patterns from EpiCare apply; still protect **PII** in communications ingest and title documents.  
- Tenant isolation on all Convex queries.  
- Audit log for evidence and commercial events.  
- External engine URLs and API keys via `packages/env-config` pattern.

---

## Related documents

- [ARCHITECTURE.md](../architecture/ARCHITECTURE.md)  
- [PIPELINES.md](../architecture/PIPELINES.md)  
- [OPEN_SOURCE_DESIGN_STACK.md](../design/OPEN_SOURCE_DESIGN_STACK.md)  
