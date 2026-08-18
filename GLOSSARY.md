# SAP RAP Glossary

Canonical terminology established through the SAP RAP lessons in this workspace.

## Contracts and runtime

**Behavior definition**:
The declared transactional contract of a RAP business object, including its operations, actions, rules, locking, authorization mode, and lifecycle behavior.
_Avoid_: Controller definition, service implementation

**Behavior projection**:
A consumer-specific behavior contract that exposes selected operations from a base business object.
_Avoid_: Behavior copy, UI behavior

**Service definition**:
A protocol-independent selection and naming of CDS entities that form a business service.
_Avoid_: OData endpoint, service implementation

**Service binding**:
The repository object that binds a service definition to a service category and protocol such as OData V4.
_Avoid_: Behavior binding, projection binding

**Transactional buffer**:
The RAP-managed transaction state containing requested changes before a successful save sequence makes them durable.
_Avoid_: Database cache, draft table

**Business invariant**:
A rule that must remain true regardless of whether the business object is invoked through Fiori elements, OData, EML, a test, or another consumer.
_Avoid_: UI rule, button rule
