# Product scope — Property Development Intelligence (PDI)

**Status:** Product concept / funding-ready specification  
**Geography (v1):** Nairobi and Kiambu counties, Kenya  
**Typology (v1):** Residential apartments and small-to-medium gated estates  

---

## Executive summary

PDI is a **development decision engine**, not a property chatbot or a standalone architecture generator. It evaluates land and proposed developments using a **shared domain model**, **deterministic engines** (financial, physical, regulatory), **specialist agents** (interpretation and investigation only), and **evidence-backed reporting** with separate feasibility and evidence-confidence scores.

The platform must:

1. Investigate land, regulation, market, and communications.
2. Generate or ingest **structured design proposals** (Pipeline A).
3. Run **viability assessments** on a fixed proposal without regenerating design (Pipeline B).
4. Challenge developer assumptions and produce investment recommendations with defensible confidence.

**Moat:** property ontology + Kenyan regulatory/market intelligence with provenance + reproducible engines + advisory layer that learns from outcomes over time.

---

## Objectives

The product behaves like a combined capability set:

- Real estate economist
- Property development consultant
- Quantity surveyor (cost intelligence, not certification)
- Financial modeller
- Urban planner (contextualised neighbourhood quality, not European copy-paste)
- Property valuer (market vs residual vs negotiation entry—distinct outputs)
- Kenyan land-use and regulatory research analyst
- Investment committee advisor

Outputs map to decision classes:

| Decision | Meaning |
|----------|---------|
| **Viable** | Proceed subject to normal professional sign-off |
| **Conditional** | Proceed after listed conditions |
| **Not viable** | Do not proceed on current economics/constraints |
| **Insufficient evidence** | More investigation required; hard stops may apply |

---

## Goals and non-goals

### Goals

- Optimise risk-adjusted development value within legal, physical, capital, and market constraints.
- Support **alternative scenarios** (unit mix, mixed-use, amenity trade-offs).
- Kenyan **county-aware** regulatory knowledge with traceable rules.
- **Immutable commercial event ledger** for negotiations (no silent overwrites of asking prices).
- **Separate scores:** development feasibility attractiveness vs evidence confidence (never blended into one “LLM confidence”).
- Investment-committee-grade PDF/report with traceable numbers.
- Modular design stack (**open-source first**) behind a semantic `DesignProposal` contract.

### Non-goals

- Replacing architects, QS, advocates, or planners for **certification or approval**.
- Inferring zoning, density, or height from neighbouring buildings.
- Presenting negotiation entry price as objective market valuation.
- Construction-ready drawings (conceptual / appraisal package only).
- Financing the construction of a private development (company/grant funding is for **the platform**, not the building).
- Single mega-agent owning all calculations.

---

## Primary users (decision log)

| Question | Recommended default (v1) | Notes |
|----------|--------------------------|-------|
| Primary user | Property developers and land investors | Expand to capital providers and consultants in R2+ |
| Agent responsibility | **Advisor** — assess, recommend, challenge | Not project execution operator in v1 |
| Urban planning depth | **Evaluate** allocation, green space, amenities; **generate** parametric layout options (not full masterplan CAD) | Neighbourhood quality score alongside financial score in R3 |
| Geography | Kenya only; Nairobi + Kiambu first | East Africa expansion after county packs mature |
| Property types | Residential apartments + gated estates | Commercial/industrial taxonomy in ontology, engines later |
| Autonomy | **Human-in-the-loop** for legal/planning/cost sign-off; autonomous analysis with escalation on hard stops | Professional review flags on report |

---

## Intelligence domains

Eight interconnected domains (shared property model throughout):

| Domain | Responsibility |
|--------|----------------|
| **Central advisory** | Orchestration, decision gates, questions engine |
| **Land intelligence** | Location, title, parcel, access, acquisition pricing |
| **Regulatory intelligence** | Zoning, building, environmental law (Kenya) |
| **Market intelligence** | Demand, comparables, rents, absorption |
| **Urban planning** | Density, amenities, walkability, contextual neighbourhood model |
| **Development design** | Property mix, layout, capacity (Pipeline A) |
| **Cost intelligence** | QS categories, rates, infrastructure |
| **Financial intelligence** | GDV, cash flow, IRR, funding, residual land value |
| **Risk intelligence** | Sensitivity, scenarios, confidence, hard stops |

See [ARCHITECTURE.md](./ARCHITECTURE.md) for engines vs modules vs agents.

---

## Property development ontology (summary)

Hierarchical taxonomy—not flat “residential”:

- **Residential** — studios through senior living, student, co-living, affordable, gated communities
- **Commercial** — retail, offices, mixed-use, hospitality, medical, co-working
- **Industrial / logistics** — warehouses, light industrial, cold chain, data centres
- **Institutional** — schools, hospitals, community facilities
- **Land / urban communities** — subdivision, TOD, regeneration, integrated neighbourhoods

Each type carries distinct economics, design constraints, market indicators, and regulatory hooks. Full tree: [ONTOLOGY.md](./ONTOLOGY.md).

---

## Land intelligence (mandatory first stage)

Land assessment precedes hypothetical return calculations.

**Inputs:** title deed/search, survey/mutation, GPS, listing, maps link, photos, landowner description, asking price, development concept.

**Dimensions:** ownership, tenure, encumbrances, verified area, shape, topography, soil (indicative), access, utilities, neighbours, environmental, planning permitted uses.

**Three price types (never conflate):**

| Type | Definition |
|------|------------|
| **Market value** | Reasonable worth in current market (evidence-based) |
| **Residual land value** | What the proposed development can afford to pay |
| **Negotiation entry price** | Recommended opening offer / ceiling strategy—not a valuation |

Acquisition module investigates comparables, listing staleness, asking vs transacted spread, seller urgency, constraints, alternative developments supporting different land prices.

---

## Urban planning differentiation

Contextualised **compact neighbourhood** model (2–5 acre estates): balance residential revenue with green space, circulation, retail, parking, utilities, stormwater—quantify amenity opportunity cost in the financial engine.

Multi-objective utility (illustrative):

\[
U = f(R, A, W, M, E, C)
\]

\(R\) return, \(A\) amenity, \(W\) walkability, \(M\) market fit, \(E\) environmental performance, \(C\) community utility.

Land-area allocation (e.g. 3-acre illustration) is **not** a substitute for footprint, GFA, parking, and statutory open-space compliance calculations.

---

## Kenyan regulatory intelligence

Maintained knowledge system—not documents-only RAG.

**Layers:** national legislation, county legislation, spatial plans, development control, building codes (incl. National Building Code 2024), environmental (NEMA/EIA), NCA, land registration, sectional properties, county finance.

**Rule record fields:** Rule ID, jurisdiction, authority, subject, rule type, legal source, effective date, applicable development types, requirement, evidence, status (verified / provisional / outdated), interpretation, review requirement (registered planner/advocate).

**County variation is essential** (e.g. Nairobi vs Kiambu). Do not assume Ruaka rules apply county-wide.

Legal agent **flags** for professional review; does not provide binding clearance.

---

## Feasibility engine (summary)

Deterministic appraisal across:

- **Physical feasibility** — developable envelope, GFA, NSA, unit ceiling, parking, amenities
- **Product mix optimisation** — scenario comparison (studios / 1-bed / 2-bed / commercial ground floor)
- **Construction cost** — categorised BOQ, spec tiers, sourced rates with confidence
- **Financial** — build-to-sell, build-to-hold, mixed; GDV, TDC, IRR, NPV, cash shortfall warnings
- **Funding** — equity, debt, JV, off-plan, mezzanine; drawdown timing

Details in [ARCHITECTURE.md](./ARCHITECTURE.md) and [PIPELINES.md](./PIPELINES.md).

---

## Communications intelligence

Ingest WhatsApp, email, listings, voice transcripts, offers, QS/architect quotes.

Extract commercial facts into an **immutable event ledger** (price changes append events, never overwrite).

Reconcile seller/broker claims against documents, market data, and model assumptions. Distinguish fact / claim / assumption / derived in the data model.

---

## Confidence framework

**Two independent scores:**

1. **Development feasibility score** — economic attractiveness of the project  
2. **Evidence confidence score** — reliability and completeness of inputs  

**Evidence weights (initial defaults):**

| Area | Weight |
|------|--------|
| Land ownership and title | 15% |
| Planning and legal compliance | 20% |
| Market and comparables | 15% |
| Construction cost evidence | 15% |
| Financial assumptions | 15% |
| Physical and environmental | 10% |
| Communications | 5% |
| Infrastructure and utilities | 5% |

**Hard stops** override favourable financials: unverified ownership, serious title dispute, apparently prohibited use, no legal access, material environmental blockers, demonstrably insufficient funding.

**Questions engine:** prioritise missing data by  
\(\text{ImpactOnDecision} \times \text{Uncertainty} \times \text{CostOfBeingWrong}\).

---

## Advisory report (principal product output)

Structured for investment committees—not chat prose.

1. Executive investment assessment (returns + evidence confidence)  
2. Acquisition recommendation (asking vs entry vs ceiling)  
3. Key risk findings (severity)  
4. Recommended actions and professional review requirements  
5. Alternative scenarios, map, conceptual allocation, financial schedules  
6. Clickable provenance on material numbers  

---

## Competitive landscape

| Capability | TestFit | Archistar | Autodesk Forma | Finch 3D | ARGUS Developer | PDI vision |
|------------|---------|-----------|----------------|----------|-----------------|------------|
| Site feasibility | Strong | Strong | Strong | Strong | Limited | Strong |
| Financial modelling | Strong | Some | Limited | Some | Strong | Strong |
| AI design generation | Strong | Strong | Growing | Strong | No | Strong (OSS-backed) |
| CAD/BIM integration | Yes | Varies | Yes | Yes | No | IFC/GeoJSON contract |
| Land / negotiation intel | Limited | Some | No | No | Valuation-focused | **Core** |
| Comms intelligence | No | No | No | No | No | **Core** |
| Kenyan legal intelligence | No | No | No | No | No | **Core** |
| Local market comparables | Limited | Limited | Limited | Limited | Data-dependent | **Core** |
| Estate / amenity optimisation | Yes | Some | Yes | Layout-focused | No | **Core** |
| Evidence confidence | No | No | No | No | Assumptions-focused | **Core** |
| IC-style reporting | Yes | Yes | Design-focused | Limited | Yes | Yes |

**Conclusion:** Category exists (TestFit + Archistar combine feasibility + generative design). **Gap:** unified Kenya-specific regulatory + acquisition + evidence + neighbourhood-quality advisory with design/finance **pipeline separation**.

Integrate paid tools optionally later; default **open-source geometry path**: [OPEN_SOURCE_DESIGN_STACK.md](./OPEN_SOURCE_DESIGN_STACK.md).

---

## User journey (guided investigation)

1. Create development project (parcel, listing, or acquisition proposal)  
2. Establish land and ownership facts (uploads, gaps)  
3. Regulatory screening (jurisdiction, instruments, environment)  
4. Market investigation (comps, demand, affordability)  
5. Generate development alternatives (Pipeline A)  
6. Build financial scenarios (Pipeline B per proposal)  
7. Challenge project (downside, contradictions, questions)  
8. Generate advisory report  

---

## Release roadmap

| Release | Capabilities | Outcome |
|---------|----------------|---------|
| **R1 — Feasibility core** | Ontology, land inputs, cost + fin engines, sensitivity, report skeleton | Working development calculator |
| **R2 — Intelligence** | Market comps, Kenyan legal RAG + rule engine v1, comms extraction, confidence | Evidence-backed advisory |
| **R3 — Spatial** | Maps, parcel geometry, zoning envelope, green space / amenity scoring, PostGIS adjunct | Planning intelligence |
| **R4 — Optimisation** | Product-mix optimisation, scenario comparison, funding structures, portfolio | Decision platform |

### Funding demonstration (6–8 weeks)

One real parcel (e.g. Ruaka / Ruiru), three concepts (apartment-led, courtyard estate, mixed-use), full Pipeline A outputs, Pipeline B on each, land acquisition memo, confidence scores, PDF report. Validate with ≥2 developers, 1 architect, 1 QS; target 2 pilot LOIs.

---

## Funding summary

Three positioning lenses from one platform—see [FUNDING_PIPELINE.md](./FUNDING_PIPELINE.md):

| Lens | Audience | Hook |
|------|----------|------|
| **A — PropTech VC** | Developers, investors | Disconnected feasibility + design workflows |
| **B — Urban impact** | County governments | Land-use intelligence, housing potential, green space |
| **C — Climate / sustainability** | Green finance | Flood, drainage, green infrastructure co-optimised with viability |

**Priority routes:** Antler East Africa (pre-seed), Mekatilili (prototype grant), Cities Deliver (city consortium; organisational), UNDP timbuktoo GreenTech, Google for Startups (credits/mentorship), ABIL (EO/satellite, future calls).

---

## Long-term vision

Natural-language portfolio queries, e.g. *“Land in Ruaka for ~40 units, max KSh 15M acquisition, utilities, green space, >20% return”* → ranked opportunities with evidence confidence.

Requires proprietary transaction comparables and outcome feedback loop (predicted vs actual costs, prices, absorption, density approvals).

---

## Related documents

- [ARCHITECTURE.md](./ARCHITECTURE.md)  
- [PIPELINES.md](./PIPELINES.md)  
- [DESIGN_PROPOSAL_CONTRACT.md](./DESIGN_PROPOSAL_CONTRACT.md)  
- [TECH_STACK.md](./TECH_STACK.md)  
- [FUNDING_PIPELINE.md](./FUNDING_PIPELINE.md)  
