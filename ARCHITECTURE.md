# Architecture — engines, modules, agents, configuration

**Principle:** Specialists and LLMs **interpret, investigate, and challenge**. Deterministic **engines** own measurable outputs. **Modules** own state and boundaries. **Configuration packs** customise behaviour without forking core logic.

Aligned with EpiCare async AI patterns: Convex jobs, Mastra webhooks, `dispatchAgentWorkflow` + polling UI ([EpiCare `mastra-async-ops`](https://github.com/bkb-Git/epi-care/blob/main/.cursor/rules/mastra-async-ops.mdc)).

---

## Layer diagram

```mermaid
flowchart TB
  subgraph config [Configurable packs]
    Ontology[property_type_taxonomy]
    CountyPack[county_rule_packs]
    RateLib[cost_and_market_rate_libraries]
    Weights[confidence_dimension_weights]
    Templates[design_brief_templates]
    Flags[feature_flags_per_release]
  end

  subgraph modules [Domain modules]
    ProjectMod[project_and_parcel]
    RegMod[regulatory_knowledge]
    MarketMod[market_evidence]
    DesignReg[design_proposal_registry]
    CommsMod[commercial_events]
    ViabilityMod[viability_runs]
    ReportMod[advisory_report]
    IngestMod[document_ingest]
  end

  subgraph engines [Deterministic engines]
    PhysEng[physical_envelope]
    PlanEng[planning_constraint]
    DesignEng[layout_optimisation]
    GeomEng[geometry_worker]
    IfcEng[ifc_quantity_extractor]
    CostEng[construction_cost]
    FinEng[financial_appraisal]
    ResidualEng[residual_acquisition]
    RuleEng[regulatory_measurable_checks]
    ConfEng[evidence_confidence]
  end

  subgraph agents [Mastra specialists]
    Orch[development_orchestrator]
    LandA[land]
    LegalA[legal]
    MarketA[market]
    PlanA[planning]
    BriefA[design_brief]
    CommsA[comms]
    CostA[cost]
    FinanceA[finance]
    RiskA[risk]
  end

  config --> modules
  modules --> engines
  agents --> modules
  agents --> engines
  Orch --> LandA
  Orch --> LegalA
  Orch --> MarketA
  Orch --> PlanA
  Orch --> BriefA
  Orch --> CommsA
  Orch --> CostA
  Orch --> FinanceA
  Orch --> RiskA
```

---

## Deterministic engines

Engines are **pure services**: versioned inputs → versioned outputs → stored calculation ledger. Unit-tested; no LLM in the numeric path.

| Engine | Input | Output | Prohibited behaviour |
|--------|-------|--------|----------------------|
| **Physical envelope** | Parcel geometry, constraint set | Max footprint, GFA ceiling, parking cap, amenity area budget | Infer density from neighbours |
| **Planning constraint** | County pack + parcel jurisdiction | Machine-readable setbacks, PR, coverage, height, parking rules | Guess permits from adjacent uses |
| **Design optimisation** | Site model, brief, templates | Pareto candidate designs (unit mix, footprints) | LLM-invented areas |
| **Geometry worker** | Optimisation result | IFC, GeoJSON, GLB, raster previews | Synchronous blocking in API routes |
| **IFC quantity extractor** | IFC asset | NSA, GFA, unit counts, parking counts | Vision-LLM on floor plans for quantities |
| **Construction cost** | Quantities, spec tier, rate library | TDC breakdown, contingency lines | Unsourced KSh/sqm |
| **Financial appraisal** | Costs, revenues, schedule, funding | GDV, profit, IRR, NPV, DSCR, cash peaks | Embedded in finance **agent** |
| **Residual / acquisition** | Fin model, market comps | Market, residual, entry, ceiling (labeled) | Entry price as “valuation” |
| **Regulatory rule check** | Proposal metrics, rule set | Pass/fail/unknown per rule ID | Legal certification |
| **Evidence / confidence** | Evidence graph, weights | Dimension scores, hard stops, question queue | Single blended “AI confidence” |

### Calculation dependency graph

When **land cost**, **permitted density**, or **unit mix** changes, dependent artefacts must invalidate or recalculate in order:

```mermaid
flowchart LR
  Parcel[Parcel_facts]
  Rules[Planning_rules]
  Proposal[DesignProposal_metrics]
  Phys[Physical_envelope]
  Cost[Construction_cost]
  Rev[Revenue_model]
  Fin[Financial_appraisal]
  Residual[Residual_land_value]
  Conf[Confidence]

  Parcel --> Phys
  Rules --> Phys
  Phys --> Proposal
  Proposal --> Cost
  Proposal --> Rev
  Cost --> Fin
  Rev --> Fin
  Fin --> Residual
  Parcel --> Conf
  Rules --> Conf
  Cost --> Conf
  Rev --> Conf
  Fin --> Conf
```

---

## Domain modules

Modules expose APIs to the dashboard, Convex mutations, and agents. They **persist state** and **invoke engines**; they do not embed financial formulas in agent prompts.

| Module | Responsibilities | Primary storage |
|--------|------------------|-----------------|
| **Project & parcel** | Projects, parcels, title/survey refs, geometry refs, acquisition context | Convex + object storage; PostGIS adjunct R3+ |
| **Regulatory knowledge** | Rule records, document ingestion, RAG chunks, verification status | Convex + `@convex-dev/rag` pattern |
| **Market evidence** | Listings, comps, rents, absorption; asking vs transacted typing | Convex |
| **Design proposal registry** | Versioned proposals, assets, validation status, immutability rules | Convex + S3-compatible storage |
| **Commercial events** | Negotiation ledger, extracted entities, reconciliation status | Convex (append-only events) |
| **Viability runs** | Links `proposalVersionId` + `assumptionBundleId` → frozen engine outputs | Convex |
| **Report** | Report templates, PDF/HTML, provenance links | Convex + export pipeline |
| **Document ingest** | OCR/PDF/DOCX extraction (reuse `@epicare/document-extract` patterns) | Convex jobs |

---

## Specialist agents (Mastra)

Agents run **async** (T1/T2). They call tools that read/write modules and **request engine runs**; they never replace engines for arithmetic.

| Agent | Pipeline | Core behaviours |
|-------|----------|-----------------|
| **Development orchestrator** | A + B | Stage gates, job routing, dependency checks, decision class |
| **Land** | A (early), B | Title/survey interpretation, gap analysis, acquisition context |
| **Legal** | A, B | Rule retrieval, plain-language explanation, professional escalation |
| **Market** | B | Comp selection narrative, demand story, absorption challenges |
| **Planning** | A | Amenity/walkability objectives, brief refinement, allocation trade-offs |
| **Design brief** | A | Developer intent → structured brief against templates |
| **Comms** | B (acquisition) | Extract events; never overwrite historical prices |
| **Cost** | B | Explain BOQ; flag QS validation needs |
| **Finance** | B | Scenario narration; all figures from FinEng |
| **Risk** | B | Contradictions, sensitivity framing, question prioritisation |

Optional **environmental design** worker (Ladybug/E+) sits after design validation in Pipeline A—deterministic indicators, agent explains trade-offs.

### Orchestrator vs pipelines

```mermaid
flowchart TB
  Orch[Orchestrator]
  subgraph pipeA [Pipeline_A_design]
    S1[Site_intelligence]
    S2[Planning_constraints]
    S3[Design_brief]
    S4[Design_generation_engine]
    S5[Design_validation]
  end
  subgraph pipeB [Pipeline_B_viability]
    V1[Regulatory_compliance]
    V2[Market]
    V3[Construction_cost]
    V4[Financial_modelling]
    V5[Acquisition_negotiation]
    V6[Risk_evidence]
    V7[Advisory_report]
  end
  Orch --> pipeA
  Orch --> pipeB
  pipeA -->|DesignProposal| pipeB
```

Pipeline B **must not** invoke design regeneration when only financial or market assumptions change.

---

## Configurable packs

Configuration is **data**, shipped as versioned packs (JSON/YAML in `packages/pdi-config` when implemented—not in this docs-only phase).

| Pack | Contents | Who edits |
|------|----------|-----------|
| **Property ontology** | Hierarchical development types, default metrics per type | Product + planner advisor |
| **County rule packs** | Rules with jurisdiction, effective dates, sources | Legal/planner curation |
| **Rate libraries** | Construction rates by location, spec, date, source, confidence | QS advisor + ingestion |
| **Market indices** | Location bands, default absorption bands (placeholder until data) | Research |
| **Financial presets** | Interest, drawdown curves, off-plan schedules, hurdle rates | User/org |
| **Confidence weights** | Dimension weights, hard-stop rule IDs | Product default; org override later |
| **Design templates** | Typology parameters for optimisation (courtyard, podium, etc.) | Architecture advisor |
| **Feature flags** | R1–R4 capability gates | Engineering |

---

## Development decision engine

Sits above viability runs:

| Input | Processing | Output |
|-------|------------|--------|
| One or more viability snapshots | Compare scenarios, apply hard stops, merge acquisition intel | `Recommendation` enum + conditions list |
| Evidence confidence | Gate “proceed” if below threshold | `Insufficient evidence` path |
| User constraints | Max land price, target IRR, timeline | Filter and rank scenarios |

The finance engine **may reject a design**: high aesthetic / low return concepts are not auto-recommended.

---

## Design principles (data architecture)

1. **Every fact has provenance** — source, date, verification level.  
2. **Every calculation has dependencies** — stored in calculation ledger.  
3. **Separate fact, assumption, derived** — never interchangeable types.  
4. **Version everything** — proposals, assumption bundles, reports.  
5. **Rules are jurisdiction-aware** — geography + effective date on every requirement.  

See [ONTOLOGY.md](./ONTOLOGY.md) and [DESIGN_PROPOSAL_CONTRACT.md](./DESIGN_PROPOSAL_CONTRACT.md).

---

## Intelligence domain map (conceptual)

Matches product scope “eight domains” to implementation layers:

| Intelligence domain | Module(s) | Engine(s) | Agent(s) |
|----------------------|-----------|-----------|----------|
| Land | Project & parcel | Residual (partial) | Land, Comms |
| Regulatory | Regulatory knowledge | Rule check, Planning constraint | Legal |
| Market | Market evidence | Financial (revenue inputs) | Market |
| Urban planning | Config templates | Physical envelope, Design optimisation | Planning, Design brief |
| Development design | Design registry | Design, Geometry, IFC | Planning, Orchestrator |
| Cost | Viability | Construction cost | Cost |
| Financial | Viability | Financial, Residual | Finance |
| Risk | Viability, Report | Confidence | Risk, Orchestrator |

---

## Related documents

- [PIPELINES.md](./PIPELINES.md)  
- [TECH_STACK.md](./TECH_STACK.md)  
- [DATA_SOURCES.md](./DATA_SOURCES.md)  
