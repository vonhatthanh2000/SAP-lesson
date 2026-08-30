# Start Here

This workspace is a one-month, evidence-driven path toward SAP RAP interview readiness.

## Today

1. Read [MISSION.md](./MISSION.md) and correct any assumption that does not match your goal.
2. Reserve a realistic weekly schedule. The roadmap assumes 18–22 focused hours; protect at least one 90-minute build block on each study day.
3. Choose an environment using the access matrix in [the primary-source research](./research/sap-rap-primary-sources.md#environment-choices-and-access-boundary). Do not make a local S/4 installation the critical path.
4. Complete [Lesson 1: Trace a RAP Request](./lessons/0001-trace-a-rap-request.html).
5. Draw the request flow from memory and answer the lesson's five retrieval questions before reading further.
6. Review the complete [one-month roadmap](./ROADMAP.md), then schedule only Week 1 on your calendar.

Current lesson: [Lesson 3 — Separate the BO Model from Its Consumer Projection](./lessons/0003-separate-bo-model-from-consumer-projection.html).

Next prepared lesson: [Lesson 4 — Expose a Read-Only OData V4 Service](./lessons/0004-expose-read-only-odata-v4-service.html). Begin it only after the Lesson 3 CDS artifacts activate and navigate correctly in ADT.

Later prepared lesson: [Lesson 5 — Shape Fiori Elements with Metadata](./lessons/0005-shape-fiori-elements-with-metadata.html). Begin it only after Lesson 4's binding, metadata, read, and navigation evidence passes.

First transactional lesson: [Lesson 6 — Enable Managed CRUD](./lessons/0006-enable-managed-crud.html). Begin it only after the Lesson 5 metadata extensions, Items facet, and value-help evidence pass.

Concurrency lesson: [Lesson 7 — Protect Edits with Draft and ETags](./lessons/0007-protect-edits-with-draft-and-etags.html). Begin it only after Lesson 6 root CRUD, Item create-by-association, and managed-numbering evidence pass.

Business-logic lesson: [Lesson 8 — Derive Amounts with Determinations](./lessons/0008-derive-amounts-with-determinations.html). Begin it only after Lesson 7 draft lifecycle and concurrency evidence pass.

Save-invariant lesson: [Lesson 9 — Reject Inconsistent Saves with Validations](./lessons/0009-reject-inconsistent-saves-with-validations.html). Begin it only after Lesson 8 derives Item and Request amounts for create, update, and delete paths.

Business-action lesson: [Lesson 10 — Control Status with RAP Actions](./lessons/0010-control-status-with-actions.html). Begin it only after Lesson 9 rejects invalid Customer, Product, and quantity states during activation.

## Course documents

- [Mission](./MISSION.md) — the outcome every lesson must serve.
- [Roadmap](./ROADMAP.md) — daily practice, portfolio evidence, video plan, and interview gates.
- [Resources](./RESOURCES.md) — curated SAP documentation and first-party samples.
- [RAP runtime map](./reference/rap-runtime-map.html) — printable architecture and responsibility reference.
- [Lesson 1 code trail](./reference/lesson-1-code-trail.html) — exact first-party SAP sample files for every Lesson 1 notion.
- [OpenSAP Travel sample analysis](./reference/opensap-travel-sample.html) — concrete end-to-end code trace and Sales Order transfer map, with historical-version warnings.
- [Interview-readiness map](./reference/interview-readiness-map.html) — maps the 160-question bank to lessons, milestones, and explicit scope boundaries.
- [SAP Technical Consultant interview bank](./reference/SAP_TechConsultant_Interview_160Q.html) — practice prompts and draft answers; verify technical claims against the primary sources above.
- [Glossary](./GLOSSARY.md) — terminology demonstrated and established during retrieval checks.
- [Primary-source research](./research/sap-rap-primary-sources.md) — detailed evidence, environment boundaries, and source map.

## First environment decision

- **No S/4 access:** use SAP BTP ABAP Environment trial/free-plan access if currently available. Build the custom RAP BO and keep S/4 integration behind a fake adapter.
- **S/4 access available:** confirm release/version compatibility before using `I_SalesOrderTP` or other standard artifacts. Treat genuine Sales integration as a stretch path, not a Week 1 prerequisite.
- **SAP Learning practice system available:** use it for guided exercises, while keeping export and public-portfolio continuity in mind.

Export your work regularly through an appropriate source-control path such as abapGit, especially when using temporary trial access.
