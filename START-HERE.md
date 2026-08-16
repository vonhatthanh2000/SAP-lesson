# Start Here

This workspace is a one-month, evidence-driven path toward SAP RAP interview readiness.

## Today

1. Read [MISSION.md](./MISSION.md) and correct any assumption that does not match your goal.
2. Reserve a realistic weekly schedule. The roadmap assumes 18–22 focused hours; protect at least one 90-minute build block on each study day.
3. Choose an environment using the access matrix in [the primary-source research](./research/sap-rap-primary-sources.md#environment-choices-and-access-boundary). Do not make a local S/4 installation the critical path.
4. Complete [Lesson 1: Trace a RAP Request](./lessons/0001-trace-a-rap-request.html).
5. Draw the request flow from memory and answer the lesson's five retrieval questions before reading further.
6. Review the complete [one-month roadmap](./ROADMAP.md), then schedule only Week 1 on your calendar.

## Course documents

- [Mission](./MISSION.md) — the outcome every lesson must serve.
- [Roadmap](./ROADMAP.md) — daily practice, portfolio evidence, video plan, and interview gates.
- [Resources](./RESOURCES.md) — curated SAP documentation and first-party samples.
- [RAP runtime map](./reference/rap-runtime-map.html) — printable architecture and responsibility reference.
- [Primary-source research](./research/sap-rap-primary-sources.md) — detailed evidence, environment boundaries, and source map.

## First environment decision

- **No S/4 access:** use SAP BTP ABAP Environment trial/free-plan access if currently available. Build the custom RAP BO and keep S/4 integration behind a fake adapter.
- **S/4 access available:** confirm release/version compatibility before using `I_SalesOrderTP` or other standard artifacts. Treat genuine Sales integration as a stretch path, not a Week 1 prerequisite.
- **SAP Learning practice system available:** use it for guided exercises, while keeping export and public-portfolio continuity in mind.

Export your work regularly through an appropriate source-control path such as abapGit, especially when using temporary trial access.
