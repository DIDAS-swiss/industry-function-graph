# Industry Function Graph

> **Origin and attribution**  
> Developed and contributed by **Accelerate GmbH** and published by DIDAS as an open-source ecosystem contribution.  
> Canonical upstream: https://github.com/Accelerate-GmbH/industry-function-graph  
> DIDAS publication: https://github.com/DIDAS-swiss/industry-function-graph

**👉 https://didas-swiss.github.io/industry-function-graph/** — the graph as a
page: the three layers, how composition is derived, every use case pattern with
its interface, the flows that realise them, and the gap register.

**Classify, connect and realise reusable ecosystem use cases.**

The Industry Function Graph separates three questions that are often mixed
together: where a use case belongs, what it requires and provides, and how it is
realised in a real ecosystem.

> Classification tells us where a use case belongs. Interfaces tell us what it
> can connect to. Realisation tells us how that use case is implemented in
> practice.

A use case is discoverable by sector, by business function and by its position
in a value stream. It exposes an interface saying what must be true before it
can run and what is true once it has. Another use case, or a whole value stream,
connects wherever its requirements are met by those outputs — computed, not
drawn by hand.

The point of building it this way is that the ecosystem keeps growing. A new
trust flow or a new credential type should find a stable place among the
concepts that already exist and connect to them, rather than forcing the
classification itself to grow another node.

[Turtle](https://didas-swiss.github.io/industry-function-graph/ifm-graph.ttl) ·
[JSON-LD](https://didas-swiss.github.io/industry-function-graph/ifm-graph.jsonld) ·
[ontology](https://didas-swiss.github.io/industry-function-graph/ontology) ·
[SHACL shapes](https://didas-swiss.github.io/industry-function-graph/ifm-shapes.ttl) ·
[composition report](./generated/composition-report.md)

```bash
curl -sO https://didas-swiss.github.io/industry-function-graph/ifm-graph.ttl
```

## The model in one picture

```
   CLASSIFICATION                INTERFACE                  REALISATION
   where does it belong?         what can connect?          how is it built?

   ┌───────────┐                                            ┌──────────────┐
   │  Sector   │◄── appliesToSector ──┐                     │     Flow     │
   └───────────┘                      │                     │ (real impl.) │
   ┌───────────┐                      │                     └──────┬───────┘
   │ Function  │◄── primaryFunction ──┤                            │
   └───────────┘                      │                     realisesUseCase
   ┌───────────┐                      │                            │
   │ValueStream│                      ▼                            ▼
   │  └ Stage  │◄── realisesStage ─┬─────────────────┬─────────────┘
   └───────────┘                   │ UseCasePattern  │
                                   └────────┬────────┘
                            requires │      │      │ provides
                                     ▼      │      ▼
                            ┌──────────┐    │   ┌──────────┐
                            │Requirement│   │   │Provision │
                            └─────┬────┘    │   └────┬─────┘
                                  │ condition    condition
                                  ▼         │         ▼
                            ┌─────────────────────────────┐
                            │         Condition           │  ◄── substantiates ──┐
                            │  (skos:broader lattice)     │                      │
                            └─────────────────────────────┘               ┌──────────────┐
                                                                          │CredentialType│
   A provision satisfies a requirement when the provided condition is     └──────────────┘
   the required one OR NARROWER. That is the whole composition rule.
```

Two rules hold the model together. Break either and it collapses back into a
taxonomy:

**A classification relationship never implies composability.** Two use cases
under the same function, in the same sector, at the same value-stream stage do
not thereby connect. They connect when one provides a condition the other
requires.

**A credential is never the plug.** A use case provides a *condition*; a
credential *substantiates* a condition; another use case requires the
*condition*, not the credential. That is what lets a second credential, issuer
or evidence mechanism serve the same business interface without any canonical
use case changing.

## 1 · Classify

Three axes, none of which says anything about what connects to what.

| Axis | Basis | What it answers |
|---|---|---|
| **Sector** | ISIC Rev. 5, NACE Rev. 2.1 at section and division level | Where the work happens. **Context, not identity** — see below. |
| **Business function** | IFM's own scheme, mapped to UNECE/Eurostat CBF and APQC PCF with SKOS mapping relations | What kind of work it is. Exactly one *primary* function per pattern. |
| **Value stream stage** | ArchiMate's `Value Stream` concept; the catalogue is IFM's own | Where the work sits end to end. |

**Sector is context.** Two flows doing the same work in different sectors
realise *one* pattern. `age-threshold-verification` is one use case with a retail
flow and a hospitality flow; `supplier-qualification` is one use case with a
pharmaceutical flow and an aerospace flow. A new sector adopting an existing
pattern adds a flow, not a use case. Where the business transformation genuinely
differs by sector, separate patterns remain right.

**Primary and supporting functions.** Exactly one primary function; supporting
functions aid discovery but never determine identity or composition. Duplicate
analysis uses the primary function and the interface, never the supporting ones.

Reach across the classification is **derived** at all three ISIC levels —
`sectionScope`, `divisionScope`, `classScope` — because "cross-sector" in
ordinary usage and "more than one ISIC section" are different claims. Banking and
insurance are different industries inside section L; pharmaceutical and aerospace
manufacturing are different industries inside section C. A single flag calls both
of those sector-specific, which is the wrong answer to the question people are
actually asking.

## 2 · Expose an interface

> A canonical use case is a reusable unit of business activity with one primary
> business function, a defined set of required conditions and one principal
> business outcome. It may expose additional outputs, but it must represent one
> coherent transformation.

```
required conditions  →  use case  →  provided conditions
```

An interface point is reified, so it can carry more than a condition id:

| Dimension | Required? | What it does |
|---|---|---|
| **condition** | yes | The concept being required or provided. |
| **subject role** | no | Which party it is about. A requirement about an organisation is not met by a provision about a person. |
| **evidence type** | no | A specific credential — *only* where the point genuinely turns on one. Most should leave it empty. |
| **context** | no | `key=value` constraints: jurisdiction, sector, actor type, governing authority, assurance, governance regime. |

Only the condition is mandatory, and an interface stays as loose as its author
left it. `validate.py` warns when a *requirement* names a specific credential,
because doing so is what stops an alternative credential from serving the same
business need.

### Conditions form a subsumption lattice

This is what makes the interfaces scale beyond exact string matching.
`secondary-education-credential-held` sits under
`education-qualification-evidence-available` via `skos:broader`, so:

- `education-qualification-issuance` provides the **narrow** condition;
- `qualification-verification` requires the **broad** one;
- the graph recognises the first as a supplier for the second.

Narrower satisfies broader, never the reverse: providing "identity evidence is
available" does not satisfy a requirement for the state e-ID specifically.

### Four kinds of condition

The distinction that matters is that **holding evidence is not the same as a
relying party having established something on it.**

| Kind | Asserts | Examples |
|---|---|---|
| `evidence` | A party holds something presentable and checkable | `eid-held`, `secondary-education-credential-held` |
| `fact` | A relying party has established something by checking | `identity-verified`, `age-threshold-established`, `legal-capacity-established` |
| `relationship` | An ongoing relationship, entitlement or access exists | `customer-relationship-open`, `access-granted` |
| `outcome` | A business decision has been taken | `supplier-qualified`, `patient-record-linked` |

Four, not more. A credential may only substantiate an `evidence` condition —
`validate.py` and the SHACL shapes both enforce it, because a credential that
could substantiate a `fact` would be a credential standing in for verification.

## 3 · Compose

`ifm:enables` is computed from the interfaces through the lattice. The
[composition report](./generated/composition-report.md) is regenerated with
everything else, so it cannot drift. Three of its chains:

```
Electronic identity held
  → identity-verification → due-diligence-onboarding → due-diligence-refresh
  → KYC attestation current

Upper-secondary qualification held
  → eid-issuance → identity-verification → qualification-verification
  → tertiary-admission → Tertiary enrolment established

Upper-secondary qualification held
  → eid-issuance → identity-verification → qualification-verification
  → employment-engagement → Employment relationship open
```

Nothing in those chains names a credential, and no row anywhere records them. A
chain is reported only if every step earns its place: drop any one and the goal
stops being reachable.

**Value streams compose too.** A stage carries its own required and provided
conditions, a use case `realisesStage` a stage, and several use cases may realise
one. The stream itself exposes start requirements and end outcomes through the
same `ifm:requires` / `ifm:provides`. `ifm:ComposableElement` is the shared
superclass, which is what lets a use case feed a stream and a stream feed a use
case without a second set of properties.

**Named dependencies are exceptional.** `ifm:requiresUseCase` exists and is
empty. "Tertiary admission requires verified qualification evidence" composes
with any upstream flow that can establish it; "tertiary admission requires the
Maturitätszeugnis issuance use case" composes with exactly one and silently
excludes every alternative. The validator errors if a named dependency duplicates
something the interface already expresses, and warns on the rest.

### Atomicity

A candidate performing several independent transformations that could be reused
on their own is not one pattern. "Employee onboarding" was one use case here; it
is now three — `identity-verification`, `qualification-verification`,
`employment-engagement` — and one flow, `employment-onboarding-ch`, composing
them. The three are reusable; that particular composition of them is not.

`validate.py` enforces the mechanical half: exactly one primary function, exactly
one principal outcome. A pattern with two principal outcomes is doing two
transformations.

## 4 · Realise

A **flow** is an actual implementation: a real sector and jurisdiction, the exact
credential types, the parties and their trust roles, the protocols and governance
it runs under, where it is documented, and its maturity and deployment evidence.

A flow **realises** one or more patterns, in order. Two consequences the model
depends on:

- **Several flows realise one pattern.** The Matura flow and the vocational-
  certificate flow both realise `education-qualification-issuance`. The retail
  and hospitality age checks both realise `age-threshold-verification`.
- **One flow realises several patterns.** `employment-onboarding-ch` realises
  three in sequence.

Keeping flows out of the taxonomy is the whole point. The ecosystem adds
implementations continuously; each should find a place among the existing
patterns rather than becoming another one.

Participants, trust roles and credentials live on the **flow**, not the pattern —
who does the work and at whose cost is exactly what differs between two
implementations of the same thing.

### Credentials are extensible by construction

`credential-conditions.csv` is many-to-many. `secondary-education-credential-held`
is substantiated by both the school-leaving certificate and the vocational
qualification certificate; neither is named by any use case. A third can be added
to that one file and every use case requiring the condition accepts it
immediately.

The graph answers, for any condition: which use cases require evidence of it,
which credential types can provide that evidence, which flows issue them, and
which downstream use cases could consume them. There are no one-to-one
assumptions between state and credential, use case and credential, or function
and credential.

## 5 · Find the gaps and the overlaps

Everything here is derived and reported, never acted on automatically.

| Report | Where |
|---|---|
| Requirements with no upstream provider | `validate.py`, [report](./generated/composition-report.md#gaps) |
| Outputs with no downstream consumer | same |
| Patterns with no implementation | same |
| Value stream stages with no use case | same |
| Credentials issued but never verified here | same |
| Overlapping patterns | `validate.py`, [report](./generated/composition-report.md#overlapping-use-case-patterns) |

**Overlap detection** goes beyond equality. Two patterns sharing a primary
function are compared on both ends of their interfaces through the lattice, and
classified `duplicate`, `specialisation`, `generalisation`, `overlap` or
`distinct`. A duplicate is an error; everything else is a warning **reported for
editorial review and never merged** — a shared shape may still be genuinely
different work. Three candidates stand at the moment, listed in the report.

## The fourteen questions

`build/queries.py` treats these as acceptance tests, not documentation. CI runs
`--check`, which fails if a query that must return something returns nothing —
so a refactor that quietly breaks composability is caught.

```bash
python3 build/queries.py            # answer all fourteen
python3 build/queries.py --check    # the CI gate
python3 build/queries.py 5 9 14     # a selection
```

1. Which use cases perform function X in sector Y?
2. Which use cases realise stage Z of value stream V?
3. What does use case A require?
4. What does use case A provide?
5. Which existing use cases can satisfy A's requirements?
6. Which use cases can consume A's outputs?
7. Which requirements currently have no upstream provider?
8. Which outputs currently have no downstream consumer?
9. Which use cases appear to overlap semantically?
10. Which real implementation flows realise a canonical use case?
11. Which credential/evidence types can satisfy a particular input requirement?
12. Which value-stream stages currently have no use case implementation?
13. Which credentials are produced but currently have no consuming flow?
14. Which composition paths connect a given starting condition to a desired outcome?

## Layout

```
industry-function-graph/
├── data/                     the source of truth — CSV, hand-edited, diffable
│   ├── CLASSIFICATION        sectors, functions, value-streams,
│   │                         value-stream-stages, use-cases
│   ├── INTERFACE             conditions, subject-roles,
│   │                         use-case-requires, use-case-provides,
│   │                         value-stream-{stage-,}{requires,provides}
│   └── REALISATION           flows, flow-realises, flow-participants,
│                             flow-credentials, credential-types,
│                             credential-conditions, trust-roles
├── ontology/
│   ├── ifm.ttl               the vocabulary
│   └── ifm-shapes.ttl        SHACL shapes for the principal RDF constraints
├── build/
│   ├── model.py              loads data/, and holds the composition engine
│   ├── build.py              data/ → generated/
│   ├── validate.py           integrity checks, run in CI
│   ├── queries.py            the fourteen acceptance queries
│   └── reconcile.py          cross-check against the Trust Flow repository
└── generated/                DO NOT EDIT — rebuilt from data/
    ├── ifm-graph.ttl         the graph, Turtle
    ├── ifm-graph.jsonld      the same graph, JSON-LD
    ├── matrix.md             the sector × function matrix, patterns and flows
    ├── composition-report.md worked chains, overlaps and gaps
    └── index.html            the browsable site, self-contained
```

## Rebuilding

No dependencies beyond Python 3:

```bash
python3 build/validate.py      # integrity checks (exit 1 on error)
python3 build/queries.py --check
python3 build/build.py         # regenerate generated/
python3 build/build.py --check # fail if generated/ is stale — what CI runs
```

`validate.py` also checks that `generated/` still matches `data/`, so a local run
cannot pass while the published graph describes the previous data.

Two checks need optional packages and skip with a warning when absent. With
`rdflib`, `validate.py` parses the generated RDF and checks that the Turtle and
the JSON-LD are the same graph. With `pyshacl` as well, it runs
[`ontology/ifm-shapes.ttl`](./ontology/ifm-shapes.ttl) over the generated graph.
CI installs both.

```bash
pip install rdflib pyshacl
```

## Adding to the graph

**A new implementation of something that already exists** — the common case —
touches no canonical concept:

1. `data/flows.csv` — the flow, its sector, jurisdiction, maturity and documentation.
2. `data/flow-realises.csv` — which pattern(s) it realises, in order.
3. `data/flow-participants.csv` and `data/flow-credentials.csv` — who and what.

**A new credential type** touches no canonical concept either:

1. `data/credential-types.csv` — format, semantic model, trust framework, profile.
2. `data/credential-conditions.csv` — which existing condition(s) it substantiates.

Every use case requiring that condition now accepts it.

**A new canonical use case** — only when the transformation itself is new:

1. `data/use-cases.csv`, then `use-case-functions.csv` (exactly one primary),
   `use-case-sectors.csv` (where it is applicable, which is broader than where it
   is implemented), `use-case-value-drivers.csv`.
2. `data/use-case-requires.csv` and `use-case-provides.csv` — the interface, with
   exactly one `principal=yes`. **Write conditions at the level of generality you
   actually need**, not the narrowest one you happen to have.
3. If a condition is genuinely missing, add it to `data/conditions.csv` with its
   `kind` and, where one exists, a `broader` condition. A condition with no
   parent and no children connects nothing.
4. Run `validate.py`, then `queries.py --check`, then `build.py`, and commit
   `data/` and `generated/` together.

Before adding one, check the overlap report. If a candidate shares a primary
function and an interface with something that exists, it is probably a flow.

## Querying

Plain SKOS plus this repository's own vocabulary, so any triple store works.

What does a use case require, and what satisfies it:

```sparql
PREFIX ifm:  <https://didas-swiss.github.io/industry-function-graph/ontology#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?useCase ?needs ?suppliedBy WHERE {
  ?uc a ifm:UseCasePattern ; rdfs:label ?useCase ; ifm:requires ?r .
  ?r ifm:condition ?c .
  ?c rdfs:label|<http://www.w3.org/2004/02/skos/core#prefLabel> ?needs .
  OPTIONAL {
    ?p ifm:satisfies ?r .
    ?supplier ifm:provides ?p ; rdfs:label ?suppliedBy .
  }
}
```

Which credentials can satisfy a requirement, through the lattice:

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT ?condition ?credential WHERE {
  ?uc ifm:requires ?r . ?r ifm:condition ?wanted .
  ?narrower skos:broader* ?wanted .
  ?cred ifm:substantiates ?narrower ; skos:prefLabel ?credential .
  ?wanted skos:prefLabel ?condition .
}
```

Which flows realise a pattern, and in which sector:

```sparql
SELECT ?pattern ?flow ?sector WHERE {
  ?f a ifm:Flow ; rdfs:label ?flow ; ifm:realisesUseCase ?uc .
  ?uc rdfs:label ?pattern .
  OPTIONAL { ?f ifm:sectorContext ?s . ?s skos:prefLabel ?sector }
}
```

## Decision-support metadata

Secondary by design, and listed last for that reason. None of it takes part in
classification or composition.

**Value drivers** — what applying a credential reduces, removes or makes
possible: `friction-reduction`, `fraud-risk-reduction`, `compliance-assurance`,
`data-quality`, `data-minimisation`, `reach-and-inclusion`, `new-revenue`. Two
are worth stating precisely: `fraud-risk-reduction` covers *selected* forms of
fraud — a credential makes forging evidence hard and does nothing about a
truthful credential presented for a fraudulent purpose; `data-minimisation`
limits disclosure to what an interaction requires, and *which* attributes it
requires is a governance question, not one the format settles.

**Transformation mode** — `digitise`, `optimise`, `redesign`, `enable`, splitting
into run and change. An IFM editorial classification, not an external standard.

**Cost and value** — recorded per participation on a *flow*.
`ifm:costValueAsymmetry` is an editorial heuristic for locating where funding or
coordination may be needed. It aggregates two yes/no judgements, weighs nothing,
and supports no inference about whether an implementation will be adopted.

**Maturity** — a property of a flow, not of a pattern. `Live` requires deployment
evidence and the validator enforces it.

**Prior evidence mechanisms** — how the same assurance is obtained today.
`ifm:reducesRelianceOn`, not "replaces": a register queried less often is still a
register, and a signed PDF whose signature the verifier can check is not the
`pdf-attachment` mechanism at all.

## Provenance

Two questions, recorded separately because they are separate:

- **`ifm:codeStatus`** — has *this concept's* notation and label been checked
  against the publication it comes from? `verified` or `provisional`.
- **`ifm:mappingStatus`** — how was a *mapping* onto an external concept arrived
  at? `editorial` or `verified`.

A mapping onto a verified concept can still be editorial. Every mapping in
`data/function-alignments.csv` is currently `editorial`; that says nothing about
the CBF or APQC concepts themselves.

| Data | `codeStatus` | Basis |
|---|---|---|
| ISIC Rev. 5 sections, divisions, classes | `verified` | Codes and titles from the official ISIC Rev. 5 structure file published by the UN Statistics Division. |
| NACE Rev. 2.1 codes | — | `ifm:naceRev21Code` at **section and division level only**, where NACE and ISIC are identical, and only for divisions the NOGA cross-check covers. |
| CBF categories | `provisional` | Labels following the UNECE/Eurostat Classification of Business Functions. No notations asserted. |
| APQC PCF categories | `provisional` | Only the cross-industry categories an alignment references, following the 13-category structure. Not redistributed here. |
| Credential types | `provisional` | None has been checked against a published schema registry. |

### What could not be verified from here

Three assertions rest on secondary material, because the primary publications are
unreachable from the environment this is maintained in:

| Assertion | Status |
|---|---|
| NACE Rev. 2.1 and ISIC Rev. 5 are identical at section and division level | Consistent with the NOGA 2025 cross-check below and with secondary descriptions of Commission Delegated Regulation (EU) 2023/137. **Not** read from the Eurostat or UNSD publication: `ec.europa.eu`, `eur-lex.europa.eu` and `unstats.un.org` are blocked by the network policy in use. |
| The CBF category labels | Seeded, `provisional`. The publication to check them against is the UN *Manual on the Classification of Business Functions*, which could not be retrieved. |
| The APQC PCF release | The 13-category cross-industry structure is what the labels follow. The point release was **not** confirmed, so `ifm:schemeVersion` records the structure rather than naming a release. |

### Identifiers, and why they drop the section letter

A division or class is `ISIC-64`, `ISIC-6419` — not `ISIC-L-64`. Section letters
move between revisions while the numbers hold: finance went K → L, education
P → Q, human health Q → R between Rev. 4 and Rev. 5, and old section J split with
the IT half becoming Rev. 5 K. Every division and class number stayed put.

Two classes *were* renumbered in Rev. 5, and anything mapped under Rev. 4 will be
wrong:

| Rev. 4 | Rev. 5 | |
|---|---|---|
| 8521 General secondary education | **8531** General secondary education | group 852 → 853 |
| 8530 Higher education | **8540** Tertiary education | renamed as well as renumbered |

### Cross-check against NOGA 2025

The sector layer was checked against the NOGA 2025 subset codified in the
[DIDAS Trust Flow Diagram Repository](https://github.com/DIDAS-swiss/Trust-Flow-Diagram-Repository):

- **22 of 22** section letters present in both, structurally identical
- **23 of 23** NOGA divisions sit under the same section in ISIC Rev. 5
- 3 section titles differ in spelling only

NOGA 2025 is the Swiss implementation of NACE Rev. 2.1, so this is consistent
with NACE and ISIC agreeing at those two levels, and means the **division number
is a usable join key** between this graph and any NOGA-classified sector.

### CBF's core/support split is not asserted over the function layer

In CBF, whether an activity is *core* or *support* depends on the enterprise, not
on the activity: issuing certificates is core for a certification body and
support for a manufacturer. Encoding that as a context-free property of an IFM
function would bake one enterprise's viewpoint into the vocabulary. So no
function is hung under `CBF-CORE` or `CBF-SUP` with `skos:broader`, the validator
rejects a `broadMatch` or `narrowMatch` onto either, and the CBF alignments that
remain are `skos:relatedMatch`. The one exception is
`service-delivery → CBF-CORE` as a `closeMatch`: CBF defines the core function as
producing the output the enterprise exists to supply, which is how
`service-delivery` is defined.

## Reconciling with the Trust Flow Diagram Repository

```bash
python3 build/reconcile.py --source ../Trust-Flow-Diagram-Repository
```

Joins on `documented_by` — now a property of **flows**, since a canonical pattern
has no documentation of its own — and compares the state interfaces and the
primary function against that repository's `sector.yaml` families. It reports
rather than fails: the two are allowed to disagree, but not to disagree
unnoticed. Needs both checkouts, so it is a local tool rather than a CI job.

## Licence

| What | Licence |
|---|---|
| The content — the mapping data in [`data/`](./data), the ontology in [`ontology/`](./ontology), everything generated from them, and the prose in this README and `NOTICE.md` | [CC BY 4.0](./LICENSE-CONTENT) |
| The software — the scripts in [`build/`](./build) and the workflows in `.github/workflows/` | [MIT](./LICENSE) |

Reuse the mapping freely, including commercially and in modified form, as long as
you attribute:

> industry-function-graph, Daniel Saeuberli, https://github.com/DIDAS-swiss/industry-function-graph

The classifications this maps onto (ISIC, NACE, CBF, APQC PCF) are published by
their respective organisations under their own terms and are **not** covered by
that grant — see [`NOTICE.md`](./NOTICE.md). Concepts marked
`ifm:codeStatus "provisional"` have not been checked against an official
publication.
