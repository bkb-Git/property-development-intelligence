# Data sources and agent data matrix

Kenya-first data strategy. **Public web access ≠ unrestricted reuse or API availability.** Every ingestion path records licence, freshness, and verification status.

---

## Priority authoritative sources

### Ardhisasa (land records)

- Title searches, registration, survey services  
- **Use:** Uploaded official searches and survey plans as high-trust evidence  
- **Caveat:** Do not assume bulk API; registration/professional access may apply  

### State Department for Lands / planning handbooks

- Physical and Land Use Planning Act, 2019  
- Development permission and control regulations  
- County development plans and guidelines  
- **Use:** Core legal-document ingestion for regulatory module + RAG  

### County planning authorities

- Nairobi City County development control, PDPs, building approval requirements  
- Kiambu County equivalents for Ruaka/Ruiru belt  
- **Use:** County rule packs; always scope rules to geography  

### National Construction Authority (NCA)

- National Building Code 2024 (in force per NCA implementation timeline)  
- Project registration requirements  
- **Use:** Building standards references; not a substitute for approved drawings  

### NEMA

- Environmental Management and Coordination Act, EIA screening  
- **Use:** Early environmental constraint flags  

### IQSK (Institute of Quantity Surveyors of Kenya)

- Building and Construction Costs Handbook, cost indices  
- **Use:** Licensed rate ingestion; verify publication rights before redistribution  

---

## Context and geospatial sources

| Source | Use | Caveat |
|--------|-----|--------|
| **OpenStreetMap** | Roads, amenities, building footprints, neighbourhood context | Incomplete; ODbL attribution; not cadastral |
| **Copernicus Data Space** | Land cover, urban growth, environmental context | Context only; not legal boundaries |
| **Council of Governors GIS** | County spatial datasets | Check dataset metadata and licences |
| **Kenya Met / climate datasets** | Sun, rainfall for environmental worker | Reference in environmental module |
| **KNBS** | Demographics, household economics | Check NC/commercial reuse on specific datasets |

---

## Market data gap

**No reliable open API for verified transaction prices.**

| Data type | Source strategy | Confidence |
|-----------|-----------------|------------|
| Asking prices | Listing portals, broker feeds | Low–medium (not transacted) |
| Transacted prices | Valuers, developers, user-contributed deals | Medium–high when documented |
| Rents | Agents, property managers, field surveys | Medium |
| Absorption | Developer disclosures, broker estimates, user feedback | Medium |

Store **asking**, **negotiated**, and **transacted** as separate evidence types. Never merge into one “market price.”

---

## Construction cost gap

- IQSK publications and licensed QS handbooks  
- Contractor quotations (user upload → extraction)  
- Historical BOQs (org-private libraries)  

Every rate record: `location`, `date`, `specTier`, `source`, `confidence`.

---

## Agent × data matrix

### Pipeline A — Design generation

| Agent / stage | Data required | Where to get it | Output |
|---------------|---------------|-----------------|--------|
| Site intelligence | Parcel polygon, dimensions, slope, access, neighbours | Survey upload, Ardhisasa, OSM, QGIS, Copernicus DEM | `SiteModel` |
| Planning constraints | Zoning, PR, coverage, height, parking, easements | County packs, Lands docs, verified plans | `ConstraintSet` |
| Design brief | Use type, mix, amenities, buyer profile | Developer input, market research | `DesignBrief` |
| Layout generation | Site + constraints + typology templates | Prior stages + config templates | Candidate designs |
| Design validation | Geometry, adjacency, areas | IFC extractor, TopologicPy, rule engine | Validation report |
| Environmental (optional) | Orientation, climate | Met data, Ladybug | Indicators |

### Pipeline B — Viability

| Agent / stage | Data required | Where to get it | Output |
|---------------|---------------|-----------------|--------|
| Regulatory compliance | Proposal metrics, rules, approval status | County authority, rule engine | Compliance risks |
| Market intelligence | Comps, rents, vacancy, competition | Portals, valuers, KNBS context | Revenue assumptions |
| Construction cost | Quantities, spec, rates | IFC quantities, IQSK, quotes | Cost plan |
| Financial modelling | Costs, revenue, schedule, funding | Assumption bundle + FinEng | Returns, cash flow |
| Acquisition | Offers, title, residual, comms | Events ledger, market, FinEng | Entry/ceiling memo |
| Risk / evidence | All evidence + calcs | Shared evidence store | Confidence, hard stops |

---

## Data collection agents (cross-cutting)

Some ingestion runs once per project and feeds multiple pipelines:

| Collector | Inputs | Feeds |
|-----------|--------|-------|
| Document ingest | Title, survey, BOQ, PDF plans | Land, regulatory, cost |
| Comms ingest | WhatsApp, email exports | Commercial events, acquisition |
| Listing ingest | URL or paste | Market claims (low trust until verified) |
| GIS enrich | Lat/lon or boundary | Site intelligence |

---

## Licensing and operational notes

1. **Ardhisasa:** Prefer user-uploaded official outputs over scraping.  
2. **KNBS:** Confirm commercial reuse per dataset licence.  
3. **IQSK:** Handbook content may require purchase/permission for automated ingestion.  
4. **OSM:** Attribution and share-alike obligations for derivative databases.  
5. **Satellite:** Copernicus free tier terms; document tile dates on evidence.  

---

## Professional verification hooks

Flag for human review when:

- Planning parameters not confirmed by instrument  
- Title not independently searched  
- Environmental screening incomplete  
- Construction rates older than configured freshness threshold  
- Seller area differs from survey by > tolerance  

---

## Related documents

- [PRODUCT_SCOPE.md](./PRODUCT_SCOPE.md)  
- [ARCHITECTURE.md](./ARCHITECTURE.md)  
- [OPEN_SOURCE_DESIGN_STACK.md](./OPEN_SOURCE_DESIGN_STACK.md)  
