# SAP RAP Resources

## Knowledge

- [SAP Help: ABAP RESTful Application Programming Model](https://help.sap.com/docs/abap-cloud/abap-rap/abap-restful-application-programming-model)
  Canonical overview of RAP's architecture and artifact responsibilities. Use for: the mental model and request lifecycle.
- [SAP Learning: Building Transactional Apps with RAP](https://learning.sap.com/courses/building-transactional-apps-with-the-abap-restful-application-programming-model/exploring-the-concept-and-architecture-of-rap)
  SAP's current structured path from architecture and behavior through EML, draft, compositions, save, events, and extensibility. Use for: the month's theory spine.
- [SAP Help: RAP Business Object Contract](https://help.sap.com/docs/abap-cloud/abap-rap/rap-business-object-contract)
  Defines the transactional contract, interaction phase, and save semantics. Use for: runtime and interview explanations.
- [SAP Help: Business Object Implementation Types](https://help.sap.com/docs/abap-cloud/abap-rap/business-object-implementation-types)
  Official distinction between managed and unmanaged implementations. Use for: architecture decisions and brownfield trade-offs.
- [SAP Help: Entity Manipulation Language](https://help.sap.com/docs/ABAP_Cloud/abap-rap/entity-manipulation-language-eml?locale=en-US)
  Type-safe local consumption of RAP business objects. Use for: tests, orchestration, and understanding `MAPPED`, `FAILED`, and `REPORTED`.
- [SAP Help: Service Definition](https://help.sap.com/docs/abap-cloud/abap-rap/service-definition) and [Service Binding](https://help.sap.com/docs/abap-cloud/abap-rap/service-binding?locale=en-US)
  Protocol-independent exposure versus attachment to OData/UI protocols. Use for: service architecture and OData V4 decisions.
- [SAP Help: Draft](https://help.sap.com/docs/abap-cloud/abap-rap/draft) and [Draft Business Object](https://help.sap.com/docs/abap-cloud/abap-rap/draft-business-object)
  Runtime-managed draft lifecycle and active/draft identity. Use for: editing lifecycle, `%tky`, and concurrency discussions.
- [SAP Help: Authorization Control](https://help.sap.com/docs/abap-cloud/abap-rap/authorization-control)
  Separates user-dependent authorization from state-dependent feature control and CDS read access. Use for: the Week 3 security model.
- [SAP Help: Side Effects](https://help.sap.com/docs/abap-cloud/abap-rap/side-effects) and [Determinations](https://help.sap.com/docs/abap-cloud/abap-rap/determinations)
  Defines metadata-driven refresh dependencies and requirements such as determination idempotence. Use for: correct rule and UI behavior.
- [SAP Help: RAP Test Concepts](https://help.sap.com/docs/abap-cloud/abap-rap/test-concepts)
  Official unit and integration testing boundaries. Use for: the portfolio test strategy.
- [SAP Help: Developing Extensions](https://help.sap.com/docs/abap-cloud/abap-cloud-background-concepts-and-overview/develop-extensions) and [Released APIs](https://help.sap.com/docs/abap-cloud/abap-development-tools-user-guide/released-apis?locale=en-us)
  Opt-in extensibility and release contracts in ABAP Cloud. Use for: clean-core and extension design.
- [SAP-samples: RAP100](https://github.com/SAP-samples/abap-platform-rap100)
  Managed greenfield workshop covering draft, numbering, determinations, validations, actions, feature control, EML, and ABAP Unit. Use for: Weeks 1–2 practice.
- [SAP-samples: RAP110](https://github.com/SAP-samples/abap-platform-rap110)
  Intermediate two-node workshop with late numbering, side effects, functions, events, and additional save. Use for: selected Week 3 exercises.
- [SAP-samples: RAP400](https://github.com/SAP-samples/abap-platform-rap-workshops/tree/main/rap4xx/rap400)
  SAP's dedicated RAP testing workshop. Use for: EML scenario tests and test isolation.
- [SAP-samples: ABAP Flight Reference Scenario](https://github.com/SAP-samples/abap-platform-refscen-flight)
  Reference data and implementation artifacts with environment-specific branches. Use for: workshop prerequisites and comparison with the portfolio domain.
- [SAP-samples: ABAP RAP Fiori Feature Showcase](https://github.com/SAP-samples/abap-platform-fiori-feature-showcase)
  Transactional draft-enabled OData V4/Fiori elements examples. Use for: UI metadata and behavior after backend correctness is established.
- [SAP-samples: ABAP Cheat Sheets](https://github.com/SAP-samples/abap-cheat-sheets)
  Focused EML and behavior-definition references with executable examples. Use for: syntax lookup, not as the curriculum.
- [SAP Tutorial: Create an ABAP Environment Trial User](https://developers.sap.com/tutorials/abap-environment-trial-onboarding.html)
  Official onboarding path for a trial ABAP Cloud environment. Use for: initial environment setup and current availability checks.
- [SAP Help: APIs for Sales](https://help.sap.com/docs/sap_s4hana_cloud/03c04db2a7434731b7fe21dca77440da/f67705a25e2b440f90a25faaffa5ffef.html)
  Official overview of S/4HANA Cloud Sales API coverage. Use for: scoping genuine SD integration only when a suitable S/4 system is available.
- [SAP Help: Sales Order EML Example](https://help.sap.com/docs/SAP_S4HANA_CLOUD/6aa39f1ac05441e5a23f484f31e477e7/d2bc96018de74001af7ad191e06b9382.html)
  First-party example of write access through `I_SalesOrderTP`. Use for: the optional local S/4 adapter only after confirming the BO is released in the target system.

## Wisdom (Communities)

- [SAP Community: Application Development](https://community.sap.com/t5/application-development/ct-p/application-development)
  SAP-hosted practitioner community. Use for: asking focused RAP design/debugging questions after including code, observed behavior, and system release.
- [SAP Community: Enterprise Resource Planning](https://community.sap.com/t5/enterprise-resource-planning/ct-p/enterprise-resource-planning)
  SAP-hosted functional community. Use for: validating whether the simplified portfolio workflow resembles real SD practice without claiming it reproduces standard SD.
- [SAP-samples organization](https://github.com/SAP-samples)
  First-party example code and issue trackers. Use for: checking version-specific workshop behavior and reporting reproducible sample issues.

## Gaps

- Access to a real S/4HANA system with configured SD scope, representative master data, roles, and released Sales APIs is not guaranteed. The core project must not depend on it.
- RAP features differ by ABAP Platform release. Confirm the backend version and select matching repository branches before importing examples.
- Trial/free-plan availability, quotas, and lifetime can change. Verify them at setup time and export work through abapGit regularly.
