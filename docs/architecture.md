# GenCodeBio — Architecture Overview (public)

This document describes *how the system is designed*, not how it is implemented. It intentionally
omits infrastructure identifiers, prompts, clinical rules and curated content.

## 1. Layers

| Layer | Responsibility |
|---|---|
| Web UI | React SPA, role-based modules, i18n (EN / PT / ES / FR), accessible clinical-instrument design system |
| API | FastAPI, layered (routers → services → models), strict input validation, audit logging |
| Jobs | Asynchronous, batched, resumable pipeline stages with explicit timeouts and cursors |
| Knowledge connectors | One adapter per external source, with caching, retry/backoff, version/provenance metadata |
| AI layer | LLM classifier constrained to cite verifiable evidence; uncited output is rejected |
| i18n | Language chosen at sign-in; stable API error codes translated client-side; a test fails if any locale is missing a key or a code |
| Data | PostgreSQL for catalog/clinical metadata; object storage for large files; analytical store planned |
| Governance | RBAC, MFA, audit trail, retention policy, LGPD/GDPR records, risk management |

## 2. Ports and adapters (cost-zero → scale)

Every infrastructure concern is an interface with swappable implementations selected by a runtime
profile:

| Profile | Queue | Files | Database | Compute |
|---|---|---|---|---|
| `local` | in-process | local disk / local S3-compatible server | PostgreSQL (container) | local containers (workflow engine planned) |
| `cloud-lite` | managed task queue | cloud object storage | managed PostgreSQL | serverless containers |
| `scale` | managed queue / streaming | S3-compatible | managed PostgreSQL | Slurm / Kubernetes / batch |

The goal: **no rewrite when moving between profiles.** The job queue, object storage, e-mail, the LLM classifier and the knowledge sources already follow this pattern, each with a local and a cloud implementation selected by configuration. Anything that would send data off the machine (for example a fallback from a local source to a public API) is an explicit, default-off option.

## 3. Evidence integrity

1. Each external lookup stores source, build/version and timestamp.
2. The classifier receives only structured evidence collected by the platform.
3. Every cited source is checked against that evidence in code before anything is persisted.
4. Criteria codes are validated against the evidence types that are actually available.
5. Population-frequency and consequence filters run **before** any paid model call.

## 4. Local reference data and reinterpretation

- **Versioned snapshots, outside the application database.** Read-only files named by their source date, with an explicit activation pointer; switching versions is a deliberate command, never automatic.
- **Match by allele.** Reference records are matched by coordinate plus normalized REF/ALT, never by rsID alone, because one rsID can name several alleles with different clinical significance.
- **Coverage rule.** A variant absent from a *covered* region is "absent from the population database" (valid evidence); a position *outside* the covered regions is an explicit technical failure, recorded as such and retried — it can never be mistaken for absence.
- **Equivalence before use.** Replacing a public API by a local source requires a written equivalence test against the original (sample, comparator with a self-check, every divergence listed and explained).
- **Reinterpretation.** A read-only endpoint compares stored evidence with the current snapshot and flags classifications that may be stale; reclassification stays a deliberate human-triggered action.
- **Provenance.** Each report states the software image, the database versions/dates and the model and prompt hash that produced each result.

## 5. Quality & compliance posture

- Research Use Only, labelled in UI and reports.
- Quality system aligned with IEC 62304 (software lifecycle) and ISO 14971 (risk management);
  regulatory pathway analysed for Brazil (ANVISA), USA (FDA) and EU (IVDR / AI Act).
- Staged delivery with a written summary, tests and traceability per stage.

## 6. Interoperability targets

GA4GH (Beacon v2, VRS, Phenopackets, DRS, htsget), HL7 FHIR Genomics, HGVS, HPO, VCF/gVCF,
RO-Crate for workflow provenance.
