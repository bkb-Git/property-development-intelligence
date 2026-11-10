# Property development ontology

Formal semantic model for PDI. All specialists operate on the **same** property, parcel, proposal, assumptions, and evidence records.

Design principles:

1. Every fact has provenance.  
2. Every calculation has dependencies.  
3. Facts, assumptions, and derived values are distinct types.  
4. Multiple versions of proposals and assessments are preserved.  
5. Rules carry jurisdiction and effective dates.

---

## Entity tree

```text
PropertyProject
 ├── Parcel
 │    ├── Title
 │    ├── Survey
 │    ├── Location
 │    ├── Boundaries
 │    ├── Access
 │    ├── Topography
 │    └── EnvironmentalConstraints
 │
 ├── Jurisdiction
 │    ├── County
 │    ├── PlanningAuthority
 │    ├── Zoning
 │    ├── PlanningInstrument
 │    └── RegulatoryRequirements
 │
 ├── DevelopmentProposal (DesignProposal versions)
 │    ├── DevelopmentType (ontology path)
 │    ├── UnitMix
 │    ├── SiteLayout
 │    ├── Amenities
 │    ├── Infrastructure
 │    └── ConstructionSpecification
 │
 ├── MarketEvidence
 │    ├── ComparableSale
 │    ├── ComparableRent
 │    ├── Listing
 │    ├── Demand
 │    └── Absorption
 │
 ├── FinancialModel (per ViabilityAssessment)
 │    ├── AcquisitionCost
 │    ├── DevelopmentCost
 │    ├── Financing
 │    ├── Revenue
 │    ├── CashFlow
 │    └── Returns
 │
 ├── Communication
 │    ├── Person
 │    ├── Offer
 │    ├── CounterOffer
 │    ├── Quotation
 │    └── Commitment
 │
 ├── Evidence
 │    ├── Source
 │    ├── Verification
 │    ├── Confidence
 │    └── ValidityPeriod
 │
 └── FeasibilityAssessment
      ├── Scenarios
      ├── Constraints
      ├── Risks
      ├── Recommendations
      └── Decision
```

---

## Core entities (fields sketch)

### `PropertyProject`

| Field | Type | Notes |
|-------|------|-------|
| `projectId` | ID | Root tenant-scoped entity |
| `name` | string | User label |
| `orgId` | ID | Multi-tenant |
| `status` | enum | `draft \| investigating \| assessed \| archived` |
| `primaryParcelId` | ID | |
| `targetDevelopmentPath` | string[] | Ontology path |

### `Parcel`

| Field | Type | Notes |
|-------|------|-------|
| `parcelId` | ID | |
| `geometry` | GeoJSON ref | May be approximate |
| `registeredAreaM2` | number? | fact if verified |
| `claimedAreaM2` | number? | claim if seller-only |
| `tenure` | enum | `freehold \| leasehold \| unknown` |
| `countyId` | ID | Drives jurisdiction packs |

### `Evidence`

| Field | Type | Notes |
|-------|------|-------|
| `evidenceId` | ID | |
| `claimType` | enum | `fact \| claim \| assumption \| derived` |
| `sourceKind` | enum | `document \| api \| user \| engine` |
| `authoritative` | boolean | |
| `verified` | boolean | |
| `obtainedAt` | datetime | |
| `contradictsEvidenceIds` | ID[] | For confidence penalties |

### `CommercialEvent` (append-only)

| Field | Type | Notes |
|-------|------|-------|
| `eventId` | ID | |
| `eventType` | enum | `asking_price \| offer \| counter \| quotation \| commitment` |
| `amountKes` | number? | |
| `communicatedAt` | datetime | |
| `extractedFrom` | evidenceId | |
| `partyRole` | enum | `seller \| buyer \| broker \| …` |

---

## Development type taxonomy

Hierarchical—not a flat list. Example paths:

```text
Residential
 ├── StudiosAndBedsitters
 ├── Apartments
 │    ├── Affordable
 │    ├── MidMarket
 │    │    ├── OneBedroom
 │    │    └── TwoBedroom
 │    └── Premium
 ├── TownhousesMaisonettes
 ├── DetachedSemiDetached
 ├── GatedCommunities
 ├── AffordableSocialHousing
 ├── StudentAccommodation
 ├── CoLiving
 └── SeniorLiving

Commercial
 ├── RetailCentres
 ├── OfficesBusinessParks
 ├── MixedUse
 ├── HotelsServicedApartments
 ├── RestaurantsHospitality
 ├── MedicalCentres
 └── CoWorking

IndustrialLogistics
 ├── WarehousesDistribution
 ├── LightIndustrialParks
 ├── Manufacturing
 ├── ColdStorage
 ├── LastMileLogistics
 └── DataCentres

SpecialisedInstitutional
 ├── HospitalsHealthcare
 ├── SchoolsEducation
 ├── StudentHousing
 ├── RetirementCommunities
 ├── SportsRecreation
 └── ReligiousCommunity

LandDevelopmentUrban
 ├── LandSubdivisionServicedPlots
 ├── PlannedResidentialEstates
 ├── IntegratedNeighbourhoods
 ├── UrbanRegeneration
 ├── TransitOrientedDevelopment
 └── SmallScaleNewUrbanCommunities
```

Each node may define default:

- Parking ratios (indicative until county rule applies)  
- Typical spec tiers  
- Revenue model hints (sell vs hold)  
- Regulatory tags (EIA likelihood, NCA registration class)  

---

## Regulatory rule entity

| Field | Example |
|-------|---------|
| `ruleId` | `KE-NBI-ZONE-001` |
| `jurisdiction` | Nairobi County |
| `authority` | Nairobi City County |
| `subject` | Permitted land use |
| `ruleType` | Development control |
| `legalSource` | Planning instrument reference |
| `effectiveDate` | ISO date |
| `applicableDevelopmentPaths` | string[] |
| `requirement` | Structured parameter or text |
| `evidenceId` | Official document section |
| `status` | `verified \| provisional \| outdated` |
| `interpretation` | AI-generated (non-binding) |
| `reviewRequirement` | Registered physical planner |

---

## Value typing

| Type | Example | Storage |
|------|---------|---------|
| **Fact** | Registered area 1,012 sqm from survey | `Evidence` + link on entity |
| **Assumption** | Construction KSh 55,000/sqm mid-market | `AssumptionBundle` + rate library ref |
| **Derived** | Estimated construction cost KSh 55M | Calculation ledger + `derivationId` |

UI and reports must label types; derived values link to engine runs.

---

## Storage strategy (target)

| Entity class | Primary store |
|--------------|---------------|
| Projects, evidence, events, assessments | Convex |
| Heavy geometry, IFC, PDFs | Object storage (S3 / Supabase Storage) |
| Spatial overlays, parcel topology (R3+) | PostGIS adjunct keyed by `parcelId` |
| Regulatory embeddings | `@convex-dev/rag` chunks with metadata filters |

---

## Related documents

- [DESIGN_PROPOSAL_CONTRACT.md](./DESIGN_PROPOSAL_CONTRACT.md)  
- [DATA_SOURCES.md](../data/DATA_SOURCES.md)  
- [ARCHITECTURE.md](./ARCHITECTURE.md)  
