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
- [SAP-samples: OpenSAP RAP sample](https://github.com/SAP-samples/abap-platform-rap-opensap)
  Historical Travel/Booking course sample organized from read-only modeling through transactional behavior, unmanaged code, and service consumption. Use for: concrete artifact tracing only; SAP explicitly marks the exercises as no longer up to date, so validate implementation choices against current SAP Help and RAP100/RAP110.
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
- [SAP Learning: Exploring SAP S/4HANA Sales Essentials](https://learning.sap.com/courses/exploring-sap-s-4hana-sales-essentials)
  Current first-party introduction to organizational units, order-to-cash, master data, presales, shipping, billing, and analytics. Use for: the conceptual spine of the SD expansion track.
- [SAP Learning: Navigating the Order-to-Cash Process Steps](https://learning.sap.com/courses/discovering-the-basics-of-sap-s-4hana-sales/executing-sales-order-management_cad9dfbe-bafc-4ed9-ac2e-6fd8422430e4)
  Traces sales order, sourcing, delivery, goods issue, billing, payment, and document flow. Use for: Lesson 11 and all end-to-end process explanations.
- [SAP Learning: Fundamental Customizing in SAP S/4HANA Sales](https://learning.sap.com/courses/fundamental-customizing-in-sap-s-4hana-sales)
  Current first-party coverage of enterprise structure and sales configuration. Use for: verifying classic IMG-oriented secondary tutorials before making configuration claims.
- [SAP Learning: Identifying the Source of Data in a Sales Document](https://learning.sap.com/courses/fundamental-customizing-in-sap-s-4hana-sales/identifying-the-source-of-data-in-a-sales-document)
  Explains how master data, reference documents, existing document values, Customizing, and runtime control contribute to Sales Order fields. Use for: field-provenance and determination reasoning.
- [SAP Learning: Executing Sales Order Management](https://learning.sap.com/courses/executing-basic-erp-processes-with-sap-s-4hana/executing-sales-order-management)
  Explains Sales Order header, item, and schedule-line scope and connects confirmations to sourcing. Use for: the Sales document hierarchy and scope reasoning.
- [SAP Help: Relationship Between Header and Items](https://help.sap.com/docs/SAP_S4HANA_CLOUD/a376cd9ea00d476b96f18dea1247e6a5/5664b65334e6b54ce10000000a174cb4.html)
  Documents how selected header values are copied and propagated to aligned items while item differences act as exceptions. Use for: header defaults and item overrides.
- [SAP Help: Sales Order Item Schedule Line](https://help.sap.com/docs/SAP_S4HANA_CLOUD/03c04db2a7434731b7fe21dca77440da/37df44581efca007e10000000a441470.html)
  Defines a schedule line as an item subdivision by quantity and date and lists requested, confirmed, delivered, and open fields. Use for: schedule-line semantics and API-oriented field inspection.
- [SAP Learning: Running an Available-to-Promise Check](https://learning.sap.com/courses/performing-the-availability-check/running-an-available-to-promise-atp-check-in-sap-s-4hana-sales_a2b4c5e3-1618-418d-a4f6-efe5ff43f7f1)
  Explains how availability-check results become confirmed or unconfirmed schedule-line quantities and dates. Use for: connecting ATP outcomes to fulfillment commitments.
- [SAP Help: How Sales Documents Are Controlled](https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/7b24a64d9d0941bda1afa753263d9e39/c264b65334e6b54ce10000000a174cb4.html)
  First-party overview of header-, item-, and schedule-line-level controls. Use for: the three-level Sales document control model.
- [SAP Learning: Configuring a Sales Document Type](https://learning.sap.com/courses/fundamental-customizing-in-sap-s-4hana-sales/configuring-a-sales-document-type)
  Explains document categories, defaults, checks, blocks, subsequent-document proposals, and safe copying of a similar standard type. Use for: header-level process control.
- [SAP Learning: Configuring an Item Category](https://learning.sap.com/courses/fundamental-customizing-in-sap-s-4hana-sales/configuring-an-item-category-1)
  Explains item-level controls for pricing, billing, business data, text/material lines, incompletion, and schedule-line participation. Use for: item behavior and delivery-relevance nuance.
- [SAP Help: Determining Sales Document Item Categories](https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/7b24a64d9d0941bda1afa753263d9e39/dc89c95360267214e10000000a174cb4.html)
  Documents the item-category determination inputs. Use for: reconstructing the lookup before investigating custom code.
- [SAP Learning: Configuring Schedule Line Categories](https://learning.sap.com/courses/fundamental-customizing-in-sap-s-4hana-sales/configuring-schedule-line-categories)
  Explains delivery relevance, movement type, requirements transfer, availability check, procurement, and blocks. Use for: fulfillment-level control.
- [SAP Learning: Assigning Schedule Line Categories](https://learning.sap.com/courses/fundamental-customizing-in-sap-s-4hana-sales/assigning-schedule-line-categories)
  Documents exact item-category/MRP-type determination and the blank-MRP fallback. Use for: diagnosing schedule-line category selection.
- [SAP Learning: Applying the Partner Function Concept](https://learning.sap.com/courses/fundamental-customizing-in-sap-s-4hana-sales/applying-the-partner-function-concept)
  Distinguishes partner types and transaction-specific partner functions and shows how master relationships are proposed into documents. Use for: Sold-to, Ship-to, Bill-to, Payer, and partner determination.
- [SAP Learning Journey: Implementing Sales in SAP S/4HANA Cloud Public Edition](https://learning.sap.com/learning-journeys/implementing-sap-s4hana-cloud-public-edition-sales)
  Consultant-oriented implementation path including sales processes, delivery scheduling, availability, third-party sales, and other core functions. Use for: shaping the later deep-SD sequence.
- [Tutorial Campus: SAP SD Tutorial](https://www.tutorialscampus.com/sap-sd/)
  Secondary 84-page library covering classic SD vocabulary, configuration, and transaction examples. Use for: breadth and supporting examples only; route through the [complete local source map](./reference/sap-sd-source-map.html) and verify current S/4HANA claims against SAP sources.

## Practice prompts (not knowledge sources)

- [SAP Technical Consultant Interview Q&A](./reference/SAP_TechConsultant_Interview_160Q.html)
  A 160-question retrieval bank spanning ABAP, CDS, Fiori, RAP, CPI, BTP, and PI/PO. Use its questions for practice, but verify its draft answers against the primary sources above.
- [Interview-readiness map](./reference/interview-readiness-map.html)
  Maps mission-aligned prompts to lessons and portfolio evidence while identifying topics that require a separate expansion track.

## Wisdom (Communities)

- [SAP Community: Application Development](https://community.sap.com/t5/application-development/ct-p/application-development)
  SAP-hosted practitioner community. Use for: asking focused RAP design/debugging questions after including code, observed behavior, and system release.
- [SAP Community: Enterprise Resource Planning](https://community.sap.com/t5/enterprise-resource-planning/ct-p/enterprise-resource-planning)
  SAP-hosted functional community. Use for: validating whether the simplified portfolio workflow resembles real SD practice without claiming it reproduces standard SD.
- [SAP-samples organization](https://github.com/SAP-samples)
  First-party example code and issue trackers. Use for: checking version-specific workshop behavior and reporting reproducible sample issues.

## Gaps

- Access to a real S/4HANA system with configured SD scope, representative master data, roles, and released Sales APIs is not guaranteed. The core project must not depend on it.
- The Tutorial Campus SAP SD pages are broad but do not consistently establish product edition, release, or whether a classic customer-master/customizing path is preferred in current S/4HANA. Treat them as secondary material.
- RAP features differ by ABAP Platform release. Confirm the backend version and select matching repository branches before importing examples.
- Trial/free-plan availability, quotas, and lifetime can change. Verify them at setup time and export work through abapGit regularly.
