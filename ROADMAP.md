# One-Month SAP RAP Interview and Portfolio Roadmap

## Honest target

One month cannot replace the production incidents, release cycles, and stakeholder work normally implied by a true mid-level title. It can make an experienced engineer **credible for RAP interviews** by producing unusually strong evidence: a coherent application, automated tests, architecture decisions, debugging demonstrations, and clear explanations of trade-offs.

The baseline assumes **18–22 focused hours per week**. If only 10–12 hours are available, keep every item marked **Core** and drop **Stretch** items.

## The flagship project

Build a **Sales Order Change & Release Workbench**, an SD-inspired custom application rather than a reimplementation of SAP SD.

### Business story

A sales coordinator creates an order with items. The application derives totals, checks business rules, identifies fulfillment risk, and moves the order through `Draft → Submitted → Released` or `On Hold`. Authorized users can release or hold an order, and each important transition is testable and explainable.

### RAP evidence map

| Capability | Portfolio evidence |
|---|---|
| CDS data model | Header/item composition, value helps, semantic annotations, currency and quantity handling |
| Transactional behavior | Managed behavior, draft, locking, ETags, numbering, save sequence explanation |
| Business logic | Determinations, validations, actions, feature control, side effects |
| Programmatic access | EML read/modify examples and a test-data utility |
| Service/UI | Service definition, OData V4 binding, Fiori elements preview, useful UI annotations |
| Security | Instance/global authorization design and negative-path demonstration |
| Quality | ABAP Unit tests, failure-path tests, test matrix, debugging narrative |
| Engineering judgment | ADRs for managed/unmanaged choice, rule placement, draft, API boundaries, and known limitations |

### Scope guardrails

- Model sales-order-like data in custom entities; do not pretend to reproduce standard SD pricing, ATP, credit management, or document flow.
- Pick one meaningful process and make it deep. Avoid a collection of unrelated CRUD demos.
- Keep standard SAP integration behind an interface. If the available system exposes a suitable released API, add it as a stretch adapter; otherwise use a deterministic fake and document the boundary.
- Never publish credentials, proprietary system screenshots, customer data, or copied SAP source code.

## Weekly rhythm

Use the same loop each day:

1. **Retrieve (10 min):** explain yesterday's request flow or design choice without notes.
2. **Learn (30–45 min):** read one primary source for the concept needed today.
3. **Build (90–150 min):** add one vertical behavior to the flagship project.
4. **Prove (30 min):** test a happy path and at least one failure path.
5. **Publish (15 min):** commit code and update the decision/debug log.

Do not record polished videos every day. Capture rough clips and notes while building; publish at weekly milestones.

## Interview-bank loop

Use the [interview-readiness map](./reference/interview-readiness-map.html) to connect each build step to the relevant prompts in the [160-question bank](./reference/SAP_TechConsultant_Interview_160Q.html). The bank is a practice instrument, not an authoritative source. For every targeted question:

1. Answer from memory in 60–90 seconds.
2. Structure the answer as **concept → runtime consequence → design choice → portfolio evidence**.
3. Correct factual gaps with SAP Help or a commit-pinned `SAP-samples` artifact.
4. Mark a question ready only after answering it on two separate days and pointing to evidence from the project.

The one-month core targets RAP, operational CDS/Fiori, and selected ABAP/BTP prerequisites. Classic RICEFW, CPI, and PI/PO remain separate expansion tracks unless the mission is explicitly broadened.

## Week 1 — Build the RAP mental model

**Outcome:** explain the complete request lifecycle and deliver a read-only vertical slice.

| Day | Core work | Evidence |
|---|---|---|
| 1 | Set up the ABAP Cloud-capable environment and tools. Draw the RAP stack from persistence to UI. | Environment note and one-page architecture diagram |
| 2 | Model order header and item entities in CDS. Focus on keys, associations, compositions, amounts, currencies, and cardinality. | Data model plus five design explanations |
| 3 | Add projection views and explain interface versus consumption boundaries. | Diagram showing which contract may evolve independently |
| 4 | Create the service definition and OData V4 binding; preview the data. Trace one read request. | Working read-only endpoint/UI and request trace |
| 5 | Add UI/value-help annotations only where they improve the use case. Explain metadata-driven UI. | Usable list/object page, not a cosmetic redesign |
| 6 | Rebuild the request flow from memory and review failure points. | Five-minute unedited explanation |
| 7 | Publish Video 1 and run a 30-minute interview drill. | Video 1: “How a RAP request actually flows” |

**Retrieval gate:** without notes, place CDS entities, behavior, projection, service definition, binding, OData, and Fiori elements in the correct order and state the responsibility of each.

## Week 2 — Own the transactional business object

**Outcome:** deliver a managed, draft-enabled transactional business object with non-trivial rules.

| Day | Core work | Evidence |
|---|---|---|
| 8 | Complete [Lesson 6: Enable Managed CRUD](./lessons/0006-enable-managed-crud.html) for header and Items. Explain transactional buffer versus database state. | Behavior contract, CRUD evidence, and lifecycle diagram |
| 9 | Complete [Lesson 7: Protect Edits with Draft and ETags](./lessons/0007-protect-edits-with-draft-and-etags.html). Test two-user and stale-update scenarios where possible. | Draft demo and concurrency notes |
| 10 | Implement determinations for derived values such as item net amount and order total. | Before/after tests and rule-placement rationale |
| 11 | Implement validations for dates, quantities, currency consistency, and status transitions. | Message behavior and negative-path test matrix |
| 12 | Add actions such as `Submit`, `Release`, and `PutOnHold`. Encode status invariants. | Action contract, authorization expectations, tests |
| 13 | Add instance feature control and side effects where the UI needs immediate refresh. | UI state demo linked to backend state |
| 14 | Publish Video 2 and Video 3. | “Managed BO lifecycle” and “Rules: determination vs validation vs action” |

**Retrieval gate:** given a new rule, decide whether it belongs in a determination, validation, action, authorization check, feature control, or UI annotation—and defend the choice.

## Week 3 — Make it production-shaped

**Outcome:** demonstrate programmatic access, authorization, testing, observability, and an explicit integration boundary.

| Day | Core work | Evidence |
|---|---|---|
| 15 | Use EML to create/read/modify the BO from ABAP. Explain `IN LOCAL MODE` and when bypassing parts of the contract is dangerous. | Small EML utility and failure handling |
| 16 | Add global/instance authorization behavior appropriate to release and hold actions. | Authorization matrix and denied-action demo |
| 17 | Build ABAP Unit tests around invariants and action transitions. Use seams/doubles for external dependencies. | Repeatable test suite and test pyramid note |
| 18 | Debug one failed save and one failed validation. Trace messages and transactional state. | Debugging log with hypothesis → evidence → fix |
| 19 | Define an outbound integration interface. Implement a fake adapter first. | API boundary ADR and deterministic test |
| 20 | **Stretch:** connect through a released standard API available in the target system, or create a small unmanaged exercise around legacy persistence. | Integration spike with explicit limitations |
| 21 | Publish Video 4 and Video 5. | “Testing RAP business rules” and “Managed vs unmanaged: a decision, not trivia” |

**Retrieval gate:** explain the save sequence, failed/reported/mapped responses, transactional consistency, and how authorization differs from hiding a button.

## Week 4 — Package evidence and rehearse interviews

**Outcome:** turn the implementation into a portfolio an interviewer can evaluate in ten minutes.

| Day | Core work | Evidence |
|---|---|---|
| 22 | Refactor names and module seams; remove tutorial residue and secrets. | Clean repository and setup script/instructions |
| 23 | Add architecture, domain model, request flow, and behavior lifecycle diagrams. | `docs/architecture.md` and diagrams |
| 24 | Write ADRs and a limitations section. Compare managed/unmanaged and draft/non-draft choices. | At least three concise ADRs |
| 25 | Run tests, ATC/static checks available in the system, authorization checks, and a clean-install rehearsal. | Quality report with unresolved findings |
| 26 | Record a seven-minute end-to-end demo: story, architecture, behavior, failure path, tests, trade-off. | Portfolio centerpiece video |
| 27 | Conduct two mock interviews: one architecture session and one debugging/code-review session. Patch weak explanations. | Question log and corrected answers |
| 28 | Publish repository, video index, resume bullets, and a 30/60/90-day continuation plan. | Shareable portfolio landing page |

## Video series

Keep each technical video **5–8 minutes** and use the same structure:

1. One business question.
2. A diagram and prediction of runtime behavior.
3. A focused code path—not a file-by-file tour.
4. A happy path and a failure path.
5. One trade-off, one limitation, and one interview question.

Recommended episodes:

1. How a RAP request flows from Fiori elements to persistence.
2. Why composition, projection, and behavior are separate contracts.
3. Determination, validation, or action? Placing business rules correctly.
4. Draft, locking, ETags, and what concurrent editing changes.
5. Testing actions and invariants through EML.
6. Managed versus unmanaged RAP using the project as evidence.
7. Final architecture and retrospective: what is production-shaped and what is not.

## Repository shape

```text
README.md                     # Problem, demo, architecture, setup, evidence index
docs/
  architecture.md             # Request flow and dependency boundaries
  domain-model.md             # SD-inspired terms and explicit simplifications
  decisions/                  # Short ADRs
  debugging/                  # Two evidence-based investigations
  interview-questions.md      # Questions answered from this implementation
videos/README.md              # Ordered links, chapters, claims, source links
tests/README.md               # Test matrix and how to run it
```

The ABAP objects themselves may live in an abapGit-compatible package/repository structure determined by the development environment.

## Interview preparation matrix

For each topic, prepare four levels of answer:

| Level | Prompt |
|---|---|
| Concept | What problem does this RAP construct solve? |
| Runtime | What happens from request to save, including failure behavior? |
| Design | Why did you choose it here, and what alternative did you reject? |
| Evidence | Show the code, test, trace, or failure that supports the answer. |

Priority questions:

- What is the difference between an interface CDS view, projection view, and service exposure?
- What does managed RAP generate, and what responsibility remains in the behavior implementation?
- When should a rule be a validation, determination, action, feature control, or authorization check?
- How do draft, locks, and ETags address different concurrency concerns?
- What are `mapped`, `failed`, and `reported` in EML?
- When is unmanaged RAP justified, and what does it cost?
- How do you test transactional behavior without relying on the UI?
- How do clean-core restrictions and released APIs shape an extension?
- Where would real SD pricing, availability, credit, partner, and document-flow behavior enter this design?

## Definition of done

The portfolio is ready to share when a reviewer can:

- understand the business problem and architecture in two minutes;
- run or watch one complete business flow and one failure flow;
- find tests for every state transition and important invariant;
- see at least three decisions with alternatives and consequences;
- distinguish implemented features, simulated integrations, and future work;
- hear concise explanations that connect code to RAP runtime behavior;
- verify that all factual claims link to first-party SAP sources.

## Evidence basis

The sequence and capability choices are grounded in [SAP's current RAP documentation and first-party sample repositories](./RESOURCES.md). See the detailed [primary-source research](./research/sap-rap-primary-sources.md) for environment compatibility, implementation-type trade-offs, SD integration boundaries, and claim-level citations checked on 2026-08-16.
