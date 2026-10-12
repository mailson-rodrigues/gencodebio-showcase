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
| `local` | in-process | local disk / MinIO | PostgreSQL (container) | local workflow engine |
| `cloud-lite` | managed task queue | cloud object storage | managed PostgreSQL | serverless containers |
| `scale` | managed queue / streaming | S3-compatible | managed PostgreSQL | Slurm / Kubernetes / batch |

The goal: **no rewrite when moving between profiles.** Today the queue and object storage already follow this pattern (local development implementation and a cloud one); extending it to the database, the LLM and every knowledge source is the next planned step.

## 3. Evidence integrity

1. Each external lookup stores source, build/version and timestamp.
2. The classifier receives only structured evidence collected by the platform.
3. Every cited source is checked against that evidence in code before anything is persisted.
4. Criteria codes are validated against the evidence types that are actually available.
5. Population-frequency and consequence filters run **before** any paid model call.

## 4. Quality & compliance posture

- Research Use Only, labelled in UI and reports.
- Quality system aligned with IEC 62304 (software lifecycle) and ISO 14971 (risk management);
  regulatory pathway analysed for Brazil (ANVISA), USA (FDA) and EU (IVDR / AI Act).
- Staged delivery with a written summary, tests and traceability per stage.

## 5. Interoperability targets

GA4GH (Beacon v2, VRS, Phenopackets, DRS, htsget), HL7 FHIR Genomics, HGVS, HPO, VCF/gVCF,
RO-Crate for workflow provenance.
