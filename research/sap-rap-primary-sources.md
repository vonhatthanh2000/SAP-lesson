# Primary-source research: one-month SAP RAP path and SD portfolio

_Status checked: 2026-08-16. Audience: an experienced database/blockchain engineer with basic ABAP syntax who wants architectural fluency, middle-level RAP interview readiness, and a video-backed portfolio in one month._

## Executive recommendation

Use a single vertical-slice project—an **SD-style Sales Order Change/Approval workbench**—to learn RAP as a runtime and business-object model, not as a syntax catalogue. Build it first as a managed, draft-enabled custom RAP BO on SAP BTP ABAP Environment. Give it a header/item composition, business invariants, actions, instance feature control, authorization, side effects, EML-based tests, an OData V4 UI service, and a Fiori elements List Report/Object Page. Then add one deliberately small unmanaged or unmanaged-save example to demonstrate brownfield reasoning. Only integrate with a genuine S/4HANA Sales Order API if suitable S/4 access and released APIs are available.

This sequence mirrors SAP's own progression. SAP's current learning course moves from architecture and OData service definition through BO behavior, EML, concurrency, actions/messages, authority checks, create/update, draft, compositions, unmanaged save, events, and extensibility ([SAP Learning: Building Transactional Apps with RAP](https://learning.sap.com/courses/building-transactional-apps-with-the-abap-restful-application-programming-model/exploring-the-concept-and-architecture-of-rap)). The official RAP100 workshop covers managed greenfield fundamentals, draft, numbering, determinations, validations, actions, feature control, ABAP Unit, and external EML access ([SAP-samples RAP100](https://github.com/SAP-samples/abap-platform-rap100)); RAP110 then adds a two-node BO, late numbering, determine actions, side effects, functions, business events, and additional save ([SAP-samples RAP110](https://github.com/SAP-samples/abap-platform-rap110)).

A month is enough to build a credible, discussable portfolio and become interview-ready on core RAP reasoning. It is not enough to claim the production breadth of a middle SAP developer: real projects also require customer-specific SD configuration, upgrade/release constraints, operations, transports, authorizations, and experience debugging existing custom code.

## Mental model to master

RAP is SAP's end-to-end model for transactional Fiori applications and Web APIs in ABAP Cloud. Its stack separates four concerns: persistence/data, domain-specific BO behavior, business-service exposure, and the client. CDS supplies the semantically rich data model; ABAP behavior implementations supply custom logic; service definitions select the exposure model; service bindings attach protocols such as OData; Fiori elements can render a UI from service metadata and annotations ([SAP Help: RAP overview](https://help.sap.com/docs/abap-cloud/abap-rap/abap-restful-application-programming-model)).

The productive interview-level explanation is:

1. **CDS entities describe the domain and BO topology.** A root plus compositions creates an aggregate boundary—for example, Sales Order Request plus Items. Interface/base views are the reusable domain model; projection views form a consumer-specific contract.
2. **The behavior definition is the BO's typed transactional contract.** It declares CRUD, actions/functions, locks, ETags, draft, validations, determinations, authorization, feature control, numbering, and persistence mapping.
3. **The behavior pool implements only the breakouts the runtime cannot supply.** Managed BOs get transactional buffering and standard CRUD/save semantics from RAP; unmanaged BOs require the provider to implement the essential REST/transaction contract ([SAP Help: BO implementation types](https://help.sap.com/docs/abap-cloud/abap-rap/business-object-implementation-types)).
4. **EML is the protocol-independent local API.** It provides type-safe ABAP access to BO operations that are declared and implemented in the behavior definition; this is how implementations, other ABAP code, and tests consume a RAP BO without going through HTTP/OData ([SAP Help: EML](https://help.sap.com/docs/ABAP_Cloud/abap-rap/entity-manipulation-language-eml?locale=en-US)).
5. **The service layer does not own business rules.** A service definition is protocol-agnostic and selects which CDS entities are exposed; one or more service bindings attach a UI/Web API category and a protocol. This preserves the separation between domain logic and protocol ([SAP Help: service definition](https://help.sap.com/docs/abap-cloud/abap-rap/service-definition), [SAP Help: service binding](https://help.sap.com/docs/abap-cloud/abap-rap/service-binding?locale=en-US)).
6. **The RAP transaction is a contract, not ordinary table CRUD.** Changes live in a transactional buffer during the interaction phase and pass through the save sequence. Consumers must receive consistent success/failure results regardless of managed or unmanaged implementation ([SAP Help: RAP BO contract](https://help.sap.com/docs/abap-cloud/abap-rap/rap-business-object-contract)).

### Managed versus unmanaged: the interview answer

- Choose **managed** for a new BO whose persistence and CRUD fit RAP conventions. RAP provides the transactional buffer, interaction phase, and standard save behavior; the developer concentrates on business rules such as validations, determinations, and actions.
- Choose **unmanaged** when wrapping an existing transactional implementation or persistence mechanism whose buffering, locks, save, messages, and numbering already exist or must remain authoritative. The developer assumes the provider contract.
- Consider **managed with unmanaged save** or **additional save** when interaction semantics fit managed RAP but persistence or follow-up integration needs a controlled breakout. SAP's course uses unmanaged save specifically to reuse existing code for a child entity, and RAP110 includes additional save.
- Do not infer that draft is unmanaged when the BO is unmanaged: RAP's draft lifecycle is always managed by the RAP runtime for both implementation types ([SAP Help: draft business object](https://help.sap.com/docs/abap-cloud/abap-rap/draft-business-object)).

## One-month learning and production plan

Assumption: 3 focused hours on weekdays and 5–6 hours each weekend day (about 25 hours/week). Every day should end with a small artifact: an ADR/diagram, test, demoable slice, or video note. Spend roughly 70% building/debugging, 20% explaining the runtime aloud, and 10% reading syntax references.

### Week 1 — See the whole pipeline, then build the simplest vertical slice

**Learning goal:** explain every RAP artifact and the request path from Fiori/OData to a CDS entity, behavior contract, transactional buffer, save sequence, and database.

- Complete the architecture and OData-service portions of [SAP Learning: Building Transactional Apps with RAP](https://learning.sap.com/courses/building-transactional-apps-with-the-abap-restful-application-programming-model/exploring-the-concept-and-architecture-of-rap).
- Set up Eclipse/ADT and an ABAP Cloud project. Import the [ABAP Flight Reference Scenario](https://github.com/SAP-samples/abap-platform-refscen-flight), selecting its `ABAP-platform-cloud` branch for BTP/Public Cloud or the branch matching the on-premise/private-cloud ABAP release.
- Work through RAP100 exercises 1–3, but redraw the generated stack in your own words. The workshop requires current ADT and a suitable ABAP system and explicitly supports SAP BTP ABAP Environment, S/4HANA Cloud Public Edition, or S/4HANA/PCE 2022+ ([RAP100 requirements](https://github.com/SAP-samples/abap-platform-rap100)).
- Start the portfolio repository: requirements, context diagram, BO aggregate diagram, state machine, decisions, and a one-command/source-import guide. Create custom tables and CDS interface/root/item entities for `SalesOrderRequest` and `SalesOrderRequestItem`.
- Add projection views, metadata extensions, a service definition, and an OData V4 UI binding. SAP recommends OData V4 wherever possible for transactional services and notes full Fiori elements V4 support for draft-enabled scenarios ([SAP Help: service binding](https://help.sap.com/docs/abap-cloud/abap-rap/service-binding?locale=en-US)).

**Video 1 (6–10 minutes):** “RAP from request to commit.” Use one diagram, show the service-binding preview, and explain why business rules are not controller code. Do not narrate typing.

**Exit proof:** a read-only List Report/Object Page and a two-minute whiteboard explanation of CDS base/projection, service definition/binding, and OData/Fiori elements.

### Week 2 — Own the business-object transaction

**Learning goal:** reason about invariants, state transitions, concurrency, draft, and framework-supplied versus custom behavior.

- Finish RAP100 exercises on determinations, validations, actions, dynamic feature control, EML, and testing ([RAP100](https://github.com/SAP-samples/abap-platform-rap100)).
- Make the portfolio BO managed and draft-enabled. Draft bridges a stateless service with a stateful editing experience by persisting changes in draft tables; the runtime manages draft for both managed and unmanaged BOs ([SAP Help: draft](https://help.sap.com/docs/abap-cloud/abap-rap/draft)). Use `%tky`, which includes the draft indicator when applicable, rather than hard-coding active-only key assumptions ([SAP Help: draft business object](https://help.sap.com/docs/abap-cloud/abap-rap/draft-business-object)).
- Implement the SD-style rules listed in “Portfolio scope” below. Keep pricing intentionally simplified and label it as a portfolio domain model, not an implementation of SAP SD pricing.
- Add ETag/concurrency behavior and demonstrate a stale-update conflict.
- Write an ABAP EML console/test consumer that creates a header with items, calls actions, reads results, and inspects `FAILED`, `MAPPED`, and `REPORTED`.

**Video 2:** “Managed RAP is a transaction runtime, not generated CRUD.” Compare declared behavior with generated runtime responsibilities; demonstrate one validation failure, one determination, one action, and draft resume/activate.

**Exit proof:** full create/edit/activate flow, state machine enforced in the backend, and EML evidence independent of the UI.

### Week 3 — Add middle-level concerns and one brownfield seam

**Learning goal:** show that the app remains correct under roles, state-dependent operations, UI refresh behavior, tests, and legacy integration.

- Use RAP110 exercises 6–12 selectively: late numbering, validations/actions/determinations, side effects, functions, events, and dynamic feature control. RAP110 is explicitly intermediate and uses a managed two-node BO ([SAP-samples RAP110](https://github.com/SAP-samples/abap-platform-rap110)).
- Add read authorization with CDS DCL and change/action authorization in behavior methods. SAP distinguishes authorization (permission depends on the user/role) from feature control (permission depends on instance state or other non-user state); read restriction is modeled through CDS access control, while modify requests use RAP authorization ([SAP Help: authorization control](https://help.sap.com/docs/abap-cloud/abap-rap/authorization-control)).
- Add instance feature control: approve/reject only while `InReview`, edit only while `New`/`Rejected`, release only while `Approved`.
- Add side effects so changing item quantity or price refreshes item amount, header total, messages, and relevant permissions. RAP side effects publish dependencies into OData metadata so the Fiori UI knows what to reread in a stateless draft flow ([SAP Help: RAP side effects](https://help.sap.com/docs/abap-cloud/abap-rap/side-effects)).
- Test determinations as idempotent and independent; SAP states that determination ordering is not fixed and repeated execution under the same conditions must not change the result ([SAP Help: determinations](https://help.sap.com/docs/abap-cloud/abap-rap/determinations)).
- Add one narrow brownfield exercise in a separate package/branch: either an unmanaged copy of a tiny BO or managed-with-unmanaged-save behind an adapter. Document responsibilities for buffer, locks, numbering, messages, and save. Do not convert the main project to unmanaged merely to show syntax.
- Follow the [RAP400 test workshop](https://github.com/SAP-samples/abap-platform-rap-workshops/tree/main/rap4xx/rap400) for ABAP Unit patterns. Distinguish isolated unit tests from integration tests ([SAP Help: RAP test concepts](https://help.sap.com/docs/abap-cloud/abap-rap/test-concepts)); use the RAP BO test-double framework when the code under test consumes another BO ([SAP Help: managing RAP BO dependencies with ABAP Unit](https://help.sap.com/docs/abap-cloud/abap-development-tools-user-guide/managing-dependencies-on-rap-business-objects-with-abap-unit)).

**Video 3:** “Correctness beyond CRUD.” Demonstrate permissions versus feature control, side effects, an EML-based test, and the managed/unmanaged decision table.

**Exit proof:** repeatable tests for business invariants and actions, negative authorization tests where the environment permits roles, and a written brownfield integration decision.

### Week 4 — Clean core, extensibility, deployment story, and interview rehearsal

**Learning goal:** turn working code into credible engineering evidence and speak about system boundaries honestly.

- Make the custom data model and behavior explicitly extensible, then add one extension field and one behavior extension in a separate package. RAP extensibility is opt-in; base data/behavior is not extensible by default ([SAP Help: develop extensions](https://help.sap.com/docs/abap-cloud/abap-cloud-background-concepts-and-overview/develop-extensions)).
- Run relevant ABAP Unit tests and ATC checks. Explain clean core in concrete terms: ABAP Cloud uses a restricted cloud-safe language and released APIs; API release contracts distinguish system-internal C1 and remote C2 use, among others ([SAP Help: developing with ABAP Cloud in BTP](https://help.sap.com/docs/btp/btp-developers-guide/abap-cloud?locale=en-US), [SAP Help: released APIs](https://help.sap.com/docs/abap-cloud/abap-development-tools-user-guide/released-apis?locale=en-us)).
- Document packaging, transports, and deployment. A package's transport layer controls whether its objects are local or attached to CTS and their consolidation route in S/4 landscapes ([SAP Help: transport layer](https://help.sap.com/docs/abap-cloud/abap-development-tools-user-guide/transport-layer?locale=en-US&state=PRODUCTION&version=s4_hana)). Keep “publish service locally,” “transport repository artifacts,” and “deploy/integrate a Fiori app” as three separate lifecycle concepts.
- Improve the Fiori elements presentation using the official [ABAP RAP Fiori feature showcase](https://github.com/SAP-samples/abap-platform-fiori-feature-showcase), which demonstrates transactional, draft-enabled OData V4 features and provides branches for BTP/Public Cloud and supported on-premise releases.
- Record a final architecture/demo video, create a three-minute interview version, and prepare 20 question cards. Each answer should use: decision → runtime mechanics → trade-off → project evidence.

**Video 4 (10–15 minutes):** “An SD-style approval workbench built with RAP.” Include architecture, aggregate/state machine, request-to-save trace, tests, security, clean-core boundary, limitations, and future genuine S/4 integration.

**Exit proof:** public source/README and four videos, reproducible setup, screenshots, test evidence, design decisions, known limitations, and no claim that synthetic data equals production SD experience.

## Realistic SD-style portfolio scope

### Domain and aggregate

Build a custom **Sales Order Change/Approval Request**, not a clone of standard VA01 or the entire order-to-cash process.

- Root `SalesOrderRequest`: UUID/semantic display number, sales organization, distribution channel, division, sold-to party, requested delivery date, document currency, total amount, status, rejection reason, created/changed audit fields.
- Child `Item`: item number, product, requested quantity/unit, unit price, net amount, requested delivery date.
- Status: `New → InReview → Approved → Released`; `InReview → Rejected → New` after correction.
- Determinations: default status/date, assign item number, calculate item amount and aggregate header total.
- Validations: header has an item; quantity positive; date not in the past; rejection needs a reason; all item currencies/units are coherent with the simplified model.
- Actions: `Submit`, `Approve`, `Reject`, `Reopen`, and optionally `ReleaseToS4` behind an adapter.
- Dynamic features: disable illegal actions and editing according to status.
- Authorization: requester can maintain own drafts; approver role can approve/reject; release role can call the outbound integration.
- Side effects: quantity/price refresh item net amount and header total; status changes refresh actions/permissions/messages.
- Draft: interrupted editing, resume, activate, discard.
- EML: action implementation and/or a local orchestrator consumes the BO; tests prove state transitions and atomic header/item creation.

This scope uses recognizable SD concepts while remaining buildable without licensed SD customizing and master data. SAP's current S/4HANA Cloud documentation confirms that sales APIs cover sales master data, prices, sales orders/other sales documents, and billing ([SAP Help: APIs for Sales](https://help.sap.com/docs/sap_s4hana_cloud/03c04db2a7434731b7fe21dca77440da/f67705a25e2b440f90a25faaffa5ffef.html)). In an S/4 system where it is released and accessible, SAP documents write access to the Sales Order transactional interface with `MODIFY ENTITIES OF I_SalesOrderTP` ([SAP Help: Sales Order EML example](https://help.sap.com/docs/SAP_S4HANA_CLOUD/6aa39f1ac05441e5a23f484f31e477e7/d2bc96018de74001af7ad191e06b9382.html)). That is the strongest optional `ReleaseToS4` target for demonstrating local RAP BO-to-BO integration. For side-by-side BTP, select the applicable released remote API in SAP Business Accelerator Hub instead. If no S/4 sandbox exists, define an interface/adapter and use a deterministic fake; state clearly that the integration has not been proven against S/4.

Avoid pretending to reproduce standard SD pricing, ATP, credit checks, partner determination, output management, delivery, billing, tax, or incompletion procedures. Those are separate business/application competencies and depend on an S/4 system, configured scope, released artifacts, and representative master/transaction data.

## Environment choices and access boundary

| Environment | What the learner can credibly do | Important boundary |
|---|---|---|
| SAP BTP ABAP Environment trial | Current official onboarding exists for a trial user; RAP100/RAP110 point to this environment, ADT, and the Flight Reference Scenario. It is suitable for custom ABAP Cloud tables/CDS/RAP behavior, OData services, Fiori elements preview, EML, and ABAP Unit ([SAP tutorial: create an ABAP Environment trial user](https://developers.sap.com/tutorials/abap-environment-trial-onboarding.html), [RAP100](https://github.com/SAP-samples/abap-platform-rap100)). | Trial is non-productive and temporary. SAP describes BTP trial as a 90-day account for personal exploration, not production/team development ([SAP Help: trial accounts and free tier](https://help.sap.com/docs/btp/sap-business-technology-platform/trial-accounts-and-free-tier?locale=en-)). Availability/region/quota must be checked during onboarding. Export the portfolio through abapGit before expiry. |
| SAP BTP ABAP Environment free plan in an enterprise PAYG/CPEA account | Small proof-of-concept development; better continuity than a disposable trial if the learner can obtain the account/entitlement. | The instance stops nightly, has community-only support/no SLA, and is not a production substitute ([SAP Help: ABAP environment service plans](https://help.sap.com/docs/btp/sap-business-technology-platform/commercial-information)). |
| SAP Learning practice system | The official course offers a bookable dedicated hands-on system and prebuilt exercise packages ([course prerequisites](https://learning.sap.com/courses/building-transactional-apps-with-the-abap-restful-application-programming-model/exploring-the-concept-and-architecture-of-rap)). | Time/access conditions depend on the SAP Learning offering; it is excellent for the course but may not be suitable as the permanent public portfolio backend. |
| SAP S/4HANA Cloud Public Edition developer extensibility | RAP/ABAP Cloud against released S/4 business objects, local APIs, CDS views, and extension points; real organizational/master data and lifecycle practices when the tenant/roles are provided. | SAP's workshop catalogue states Public Edition developer extensibility requires a three-system landscape. Access, business roles, communication arrangements, and released API availability are customer/partner-system concerns ([SAP RAP workshops](https://github.com/SAP-samples/abap-platform-rap-workshops)). |
| SAP S/4HANA Cloud Private Edition or S/4HANA on-premise | Genuine SD artifacts/data, CTS landscape experience, and—on supported releases—ABAP Cloud plus RAP. RAP100 supports 2022+; RAP110's current intermediate content requires 2023+ ([RAP100](https://github.com/SAP-samples/abap-platform-rap100), [RAP110](https://github.com/SAP-samples/abap-platform-rap110)). | Version matters. Classic objects are not automatically allowed in ABAP Cloud. Prefer released APIs; where none exist, SAP documents a three-tier approach with wrappers, which requires the relevant S/4 system and Standard ABAP tier ([SAP Help: developing in ABAP Cloud and consuming classic APIs](https://help.sap.com/docs/abap-cloud/developer-guide-from-classic-abap-to-abap-cloud/develop)). |

Do not make “install a local S/4 developer edition” the critical path. The verified official materials above guarantee neither a universally available free local S/4 developer edition nor an SD-configured system. Build the core portfolio on BTP/learning-system access, and treat genuine SD integration as an explicit optional track.

## Primary-source map by capability

| Capability | Best primary source | What to extract/practice |
|---|---|---|
| Architecture and artifacts | [RAP overview](https://help.sap.com/docs/abap-cloud/abap-rap/abap-restful-application-programming-model), [SAP Learning RAP course](https://learning.sap.com/courses/building-transactional-apps-with-the-abap-restful-application-programming-model/exploring-the-concept-and-architecture-of-rap) | Four-layer architecture, CDS/BO/service/client separation, request path. |
| Managed/unmanaged | [BO implementation types](https://help.sap.com/docs/abap-cloud/abap-rap/business-object-implementation-types), [EML cheat sheet](https://github.com/SAP-samples/abap-cheat-sheets/blob/main/08_EML_ABAP_for_RAP.md) | Runtime/provider responsibility, transactional buffer, greenfield/brownfield decision. |
| CDS and service exposure | [Service definition](https://help.sap.com/docs/abap-cloud/abap-rap/service-definition), [service binding](https://help.sap.com/docs/abap-cloud/abap-rap/service-binding?locale=en-US) | Base/projection model, protocol independence, OData V4 UI binding and preview. |
| BDEF/BIMP and EML | [RAP BO contract](https://help.sap.com/docs/abap-cloud/abap-rap/rap-business-object-contract), [EML](https://help.sap.com/docs/ABAP_Cloud/abap-rap/entity-manipulation-language-eml?locale=en-US), [BDL cheat sheet](https://github.com/SAP-samples/abap-cheat-sheets/blob/main/36_RAP_Behavior_Definition_Language.md) | Typed BO contract, operations, `FAILED/MAPPED/REPORTED`, local consumption. |
| Draft | [Draft](https://help.sap.com/docs/abap-cloud/abap-rap/draft), [Draft BO](https://help.sap.com/docs/abap-cloud/abap-rap/draft-business-object) | Stateless/stateful bridge, active versus draft keys, activation flow. |
| Validations/determinations/actions | [Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/determinations), [action definition](https://help.sap.com/docs/abap-cloud/abap-rap/action-definition), [RAP100](https://github.com/SAP-samples/abap-platform-rap100) | Invariants versus derived data versus explicit state transitions; idempotence and messages. |
| Authorization/features/side effects | [Authorization control](https://help.sap.com/docs/abap-cloud/abap-rap/authorization-control), [side effects](https://help.sap.com/docs/abap-cloud/abap-rap/side-effects), [RAP110](https://github.com/SAP-samples/abap-platform-rap110) | User-dependent permission, state-dependent capability, metadata-driven UI refresh. |
| Testing | [Test concepts](https://help.sap.com/docs/abap-cloud/abap-rap/test-concepts), [RAP400](https://github.com/SAP-samples/abap-platform-rap-workshops/tree/main/rap4xx/rap400), [RAP BO test doubles](https://help.sap.com/docs/abap-cloud/abap-development-tools-user-guide/managing-dependencies-on-rap-business-objects-with-abap-unit) | Unit versus integration boundary, EML scenario tests, dependency isolation. |
| Extensibility/clean core | [Develop extensions](https://help.sap.com/docs/abap-cloud/abap-cloud-background-concepts-and-overview/develop-extensions), [released APIs](https://help.sap.com/docs/abap-cloud/abap-development-tools-user-guide/released-apis?locale=en-us), [clean-core extensibility](https://help.sap.com/docs/abap-cloud/developer-guide-from-classic-abap-to-abap-cloud/clean-core-extensibility-and-abap-based-extensions) | Opt-in extensions, release contracts, upgrade-stable dependencies, three-tier boundary. |
| Transport/deployment | [Transport layer](https://help.sap.com/docs/abap-cloud/abap-development-tools-user-guide/transport-layer?locale=en-US&state=PRODUCTION&version=s4_hana), [Deploy ABAP](https://help.sap.com/docs/btp/btp-developers-guide/deploy-abap), [RAP100 deployment exercises](https://github.com/SAP-samples/abap-platform-rap100) | CTS/package routes in S/4; software-component deployment and imports in BTP; local service publication versus UI deployment/launchpad integration. |

## Recommended official repositories, in order

1. [SAP-samples/abap-platform-rap100](https://github.com/SAP-samples/abap-platform-rap100) — the main first-week/second-week lab. It is compact enough to finish and covers the managed draft core plus EML and unit testing.
2. [SAP-samples/abap-platform-rap110](https://github.com/SAP-samples/abap-platform-rap110) — the main intermediate source. Mine specific exercises rather than copying the whole Travel app.
3. [SAP-samples/abap-platform-refscen-flight](https://github.com/SAP-samples/abap-platform-refscen-flight) — reusable reference data, services, and legacy logic, with branches for ABAP Platform Cloud and specific on-premise releases.
4. [SAP-samples/abap-platform-rap-workshops](https://github.com/SAP-samples/abap-platform-rap-workshops) — catalogue for RAP400 testing and RAP6xx clean-core/extensibility follow-ups.
5. [SAP-samples/abap-cheat-sheets](https://github.com/SAP-samples/abap-cheat-sheets) — targeted EML and behavior-definition lookup, including executable managed/unmanaged/draft examples. Use it as a reference, not the month-long curriculum.
6. [SAP-samples/abap-platform-fiori-feature-showcase](https://github.com/SAP-samples/abap-platform-fiori-feature-showcase) — polish and understand annotation-driven OData V4 Fiori elements behavior after backend correctness exists.
7. [SAP-samples/cloud-abap-rap](https://github.com/SAP-samples/cloud-abap-rap) — inspect the RAP generator to understand boilerplate automation and generated extensibility; do not let generation replace artifact/runtime understanding.

## Interview readiness checklist

The learner should be able to answer and demonstrate all of these without syntax memorization:

- Trace create/update/action requests across OData, projection, BO behavior, transactional buffer, validations/determinations, save, and response messages.
- Explain why a composition boundary matters for atomicity and lifecycle, and why an association is different.
- Choose managed, unmanaged, managed-with-unmanaged-save, or additional save for a stated scenario.
- Explain draft lifecycle and why `%tky` matters.
- Separate validation, determination, action, feature control, authorization, side effect, and DCL responsibilities.
- Explain EML as typed local BO consumption and interpret `FAILED`, `MAPPED`, and `REPORTED`.
- Explain lock/ETag/concurrency behavior and show a stale-update case.
- Explain why a service definition is protocol-agnostic and why OData V4 UI binding is preferred for a new draft transactional Fiori elements service.
- Describe unit versus integration tests and show business-rule tests through EML.
- Explain clean core using ABAP language version and C0/C1/C2 release contracts, not slogans.
- Explain what gets activated/published locally, what gets transported, and what is separately deployed/integrated as a Fiori app.
- State precisely what the portfolio simulates and what has—or has not—been exercised against a real S/4 SD system.

## Gaps and volatility to re-check

- Trial/free-plan region availability, quotas, lifecycle, and onboarding can change; verify immediately before starting using the official BTP and ABAP Environment pages linked above.
- RAP feature availability varies by ABAP Platform/S/4 release. The current repositories themselves impose different minimums (RAP100: 2022+, RAP110: 2023+); always select a repository branch matching the backend.
- A generic BTP ABAP system does not provide customer S/4 SD configuration/master data. The exact released Sales Order local/remote API and its version must be checked in the target system and SAP Business Accelerator Hub before implementation.
- Official sample repositories are instructional and sometimes explicitly “as-is”; they are evidence sources, not production architecture mandates.
- The research found no official primary source guaranteeing universally available, free, locally installable S/4HANA developer edition access with usable SD scope. Do not anchor the one-month plan on it.
