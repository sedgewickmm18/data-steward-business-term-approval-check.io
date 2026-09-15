# Data Steward — Business Term Approval Checks

A [reveal.js](https://revealjs.com/) presentation for Data Stewards covering the review and approval process for business glossary artifacts and a migration playbook for large-scale glossary consolidation.

> **Maintenance instruction for Bob:** After every significant change to [`index.html`](index.html) or [`docs/Large_Scale_Metadata_Migration.md`](docs/Large_Scale_Metadata_Migration.md) — new slides, restructured sections, revised examples, or renamed checks — update this README to keep the content summary accurate.

---

## Presentation structure

The presentation ([`index.html`](index.html)) is a self-contained reveal.js deck served directly from the repository root. It covers three main topics shown on the agenda slide:

### 1 · Reviewing a Business Term for Publication (3 checks)

Three quality gates a Data Steward must pass before approving or rejecting a proposed business term:

| # | Check | What to verify |
|---|-------|----------------|
| 1 | **Wording & Clarity** | Can a non-expert understand the description? Abbreviations expanded, synonyms listed, units stated, complete sentences. |
| 2 | **Ontological Fit & Conflicts** | Does the term belong in its assigned category? Are there near-duplicates or clashing terms already published (e.g. `Client Id` vs `Customer Number`)? |
| 3 | **Linkage to Existing Data** | Which database / table / column carries this data? Are data types consistent? Are mapped columns already tagged with a conflicting term? |

Each check slide follows the same layout: a *what to look for* panel on the left and a concrete worked *example* on the right (using realistic terms such as `Cap Rate`, `Net Operating Income`, and `Client Id`).

A **summary slide** ("Three Checks at a Glance") recaps all three gates.

---

### 2 · Agentic Skill — Business Term Evaluation

The dedicated `business-term-evaluation` skill automates the business-term workflow:

- **Phase 0** — Locates the business term and determines DRAFT vs PUBLISHED status.
- **Phase 1** — Wording quality check (clarity, jargon detection, improvement suggestions).
- **Phase 2** — Ontology check (category fit and clash detection against existing business terms).
- **Phase 3** — Consolidated business-term evaluation report with approve / reject / change guidance.

**MCP tools used by the business-term skill:**

| Tool | Purpose |
|------|---------|
| `list_draft_artifacts` | Discovers all pending draft artifacts; supplies `artifact_id` and `version_id`. |
| `get_artifact_details` | Retrieves full details for a DRAFT artifact (descriptions, relationships, stewards, version history). |
| `list_business_terms_by_category` | **Two calls:** (1) with description → BM25-ranks top 5 semantically similar terms (semantic clash detection); (2) without description → returns all terms in the category for exact/near-name matching. |
| `search_governance_artifacts` | Confirms published status; fallback when the category cannot be resolved for a business term. |

**Activation phrases:** *"Should I approve this business term?"*, *"Evaluate [term name]"*, *"Review [term] for approval"*, *"What do I need to know about [term]?"*

The skill definition lives in [`business-term-evaluation-skill.md`](business-term-evaluation-skill.md).

---

### 3 · Reviewing a Data Class Definition for Publication (4 checks)

Data classes classify data by column name and format (e.g. `Global Location Number (GLN)`, `Steuernummer`). Their review differs from business terms — no category hierarchy, but four specific gates:

| # | Check | What to verify |
|---|-------|----------------|
| 1 | **Uniqueness** | The name is globally unique across the full registry (no category scope to disambiguate). Check exact matches and abbreviation clashes. |
| 2 | **Fingerprint** | A regex or natural-language rule that lets classifiers recognise valid values. Must be specific enough to avoid false positives. |
| 3 | **Data Quality Rules** | Testable violation rules (format, completeness, uniqueness, referential) with severity levels (Critical / Major / Minor). |
| 4 | **Sensitivity & PII Likelihood** | Rate PII risk as High / Medium / Low. Document in the description — IKC has no dedicated PII marker field yet. |

A **summary slide** ("Four Checks at a Glance — Data Classes") closes this section and lists known limitations (columnar scope only, no dedicated PII marker, Java/JS type rules deferred).

The separate `data-class-evaluation` skill uses seven phases: status determination, description clarity, global uniqueness, column-scope fingerprint, value-level data-quality readiness, sensitivity/likely PII exposure, and a consolidated report. It uses `list_draft_artifacts`, `get_artifact_details`, and `search_governance_artifacts`; it does not use category-scoped business-term tooling. Its definition lives in [`data-class-evaluation.md`](data-class-evaluation.md).

---

### 4 · Large-Scale Glossary Migration (5 phases)

A five-phase playbook for reducing 1,000+ overlapping legacy terms to a clean, canonical glossary. Full detail is also available as a standalone document in [`docs/Large_Scale_Metadata_Migration.md`](docs/Large_Scale_Metadata_Migration.md).

| Phase | Name | Goal |
|-------|------|------|
| 1 | **Inventory & Assessment** | Export all terms, profile data quality, establish a baseline count. |
| 2 | **Hybrid Deduplication & Semantic Clustering** | Combine lexical similarity and AI semantic embeddings to collapse duplicates; target a 40–70 % reduction before touching definitions. |
| 3 | **Clarity & Definition Standardisation** | Enforce ISO 11179-style templates; automate quality checks (short, circular, acronym-heavy definitions); route failures to SMEs. |
| 4 | **Ontological Alignment** | Map surviving terms to top-level ontology classes using `isA`, `partOf`, `hasProperty` predicates; validate with a semantic reasoner. |
| 5 | **Governance & Load — Lock the Gate** | Load canonical terms into the production catalog; enforce the three business-term quality gates via the Data Intelligence Business Term Approval workflow template. |

---

## Repository layout

```
.
├── index.html                          # Main reveal.js presentation (all slides)
├── business-term-evaluation-skill.md   # Bob skill — business-term evaluation
├── data-class-evaluation.md            # Bob skill — column-oriented data-class evaluation
├── docs/
│   └── Large_Scale_Metadata_Migration.md  # Standalone migration playbook (prose)
└── README.md                           # This file
```

---

## Running the presentation locally

The presentation uses reveal.js **from CDN** — no build step required.

```bash
# Any static file server works, e.g.:
python3 -m http.server 8080
# then open http://localhost:8080 in a browser
```

Keyboard shortcuts (standard reveal.js):

| Key | Action |
|-----|--------|
| `→` / `Space` | Next slide |
| `←` | Previous slide |
| `S` | Open speaker notes |
| `O` | Slide overview |
| `F` | Full screen |
| `?` | All keyboard shortcuts |

---

## Context for Bob tasks

When working on this repository, keep the following in mind:

- **Presentation source** is entirely in [`index.html`](index.html) — a single-file reveal.js deck with inline CSS and no build pipeline.
- **Slide sections** are identified by `id` attributes (`#wording`, `#ontology`, `#data`, `#agentic-skill`, `#dc-title`, `#dc-uniqueness`, `#dc-fingerprint`, `#dc-quality`, `#dc-sensitivity`, `#dc-summary`, `#dc-agentic-skill`, `#migration-title`, `#migration-phase1` … `#migration-phase5`).
- **Two distinct agentic skills** drive evaluation: [`business-term-evaluation-skill.md`](business-term-evaluation-skill.md) covers business-term wording and ontology checks, while [`data-class-evaluation.md`](data-class-evaluation.md) covers global uniqueness and column-oriented data-class evaluation. Keep the corresponding *Agentic Skill* slides (`#agentic-skill` and `#dc-agentic-skill`) aligned with their respective skill definitions.
- **README maintenance:** Update this file whenever slides are added or removed, check criteria change, the skill phases change, or the migration playbook is revised.
