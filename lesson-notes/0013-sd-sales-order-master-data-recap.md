# Lesson 13 Recap: Trace Sales Order Data to Its Source

## Lesson objective

Given a field in a Sales Order, explain:

1. which object supplied it,
2. the organizational scope of that source,
3. whether the value was entered, copied, proposed, derived, or overridden,
4. what the value means in this specific transaction, and
5. whether a later master-data change affects the existing order.

The goal is to replace the vague explanation “SAP filled it automatically” with a precise account of data provenance.

## The lesson in one sentence

A Sales Order is a transactional snapshot assembled from reusable master data, reference documents, Customizing, existing document values, and user input; each field has its own source and determination rules.

## The central answer frame

Use this sequence whenever an interviewer asks where a Sales Order value comes from:

```text
Field
  → source object
  → organizational scope
  → copy / proposal / determination
  → value stored in the transaction
  → effect of a later source change
```

Example:

```text
Payment Terms
  → Sold-to/Payer customer master data
  → relevant organizational scope
  → proposed during order creation
  → stored as Sales Order data
  → later master-data changes normally do not replace it automatically
```

Do not assume one universal precedence order. The responsible source and priority are field-specific and can depend on the document type, reference document, Customizing, existing values, and SAP release.

## Main sources of Sales Order data

```text
Reference document
        +
Master data
        +
Customizing
        +
Existing document values
        +
User input
        ↓
   Sales Order
```

| Source | Example contribution |
|---|---|
| Reference document | A quotation supplies copied partners, items, quantities, or commercial terms. |
| Master data | Business Partner, Material, pricing, and customer-material records supply reusable facts and defaults. |
| Customizing | Defines controls, allowed processing, and determination strategies. |
| Existing document data | Values already present become inputs to later determinations. |
| User input | A user enters or changes a value when field control and business rules permit it. |

## Master data versus transactional data

| Master data | Transactional Sales Order data |
|---|---|
| Reusable facts about a party, product, or business relationship | Facts accepted for one specific sale |
| Reused by many transactions | Belongs to one document and its history |
| Maintained for defined organizational scopes | Copied, proposed, derived, or entered in the order context |
| Has an independent lifecycle | Follows the Sales Order lifecycle |
| Can change as the real-world business changes | Normally preserves the decision made for that transaction |

The Sales Order is not a live mirror of every master record. Once values are copied or proposed and accepted, they normally become the order's own transactional data.

```text
Master data change ≠ automatic rewrite of an existing Sales Order
```

New documents normally use the latest applicable master data. Existing documents normally retain their copied values unless they are explicitly changed or specialized SAP behavior applies. Customer addresses, including time-dependent address handling, are a notable exception.

## Do not confuse three kinds of “role”

| Concept | Question answered | Examples |
|---|---|---|
| BP Category | What fundamental kind of party is it? | Person, Organization, Group |
| BP Role | In which application context can it operate, and which data segments are required? | Customer, FI Customer, Supplier |
| Partner Function | What responsibility does it have in this sales transaction? | Sold-to, Ship-to, Bill-to, Payer |

Memory pattern:

```text
Category = what it is
Role     = where it operates
Function = what it does in this transaction
```

## Business Partner organizational scope

| Data level | Purpose | Typical examples |
|---|---|---|
| General / client-wide | Shared identity across application contexts | Name, core address, contact and control data |
| Company Code | Accounts Receivable and financial processing for one accounting unit | Reconciliation account, payment and correspondence data |
| Sales Area | Sales, shipping, and billing relationship for one Sales Org/Channel/Division | Sales currency, shipping conditions, payment terms, Plant proposal, partner relationships |

A Business Partner may exist at the general level but still be unusable in a Sales Order if it lacks the required customer data for that Sales Area. Billing can also require the relevant Company Code data for the Payer.

## The four customer partner functions

| Partner function | Responsibility | Data emphasis |
|---|---|---|
| Sold-to Party | Places or owns the commercial order relationship | Sales relationship and commercial defaults |
| Ship-to Party | Receives the goods | Delivery address, unloading point, receiving hours |
| Bill-to Party | Receives the invoice | Invoice address and communication |
| Payer | Settles the receivable | Payment-related and financial data |

One Business Partner may perform all four functions. They may also be split across different partners. For example:

```text
Headquarters       → Sold-to
Regional warehouse → Ship-to
Accounts payable   → Bill-to
Treasury company   → Payer
```

Partner determination proposes the applicable partners and controls whether each function is allowed, required, changeable, or determined at header or item level.

## Material master: one product with scoped views

The Material master represents one reusable product. Different views and organizational scopes let departments maintain the data they need without creating unrelated copies of the product.

| Scope or view | Question answered | Examples |
|---|---|---|
| General | What is this product? | Description, base unit, product attributes |
| Sales Organization + Distribution Channel | How is it sold through this Distribution Chain? | Sales unit and sales-processing data |
| Plant | How is it planned, supplied, and handled at this location? | Availability, MRP, loading and fulfillment data |
| Accounting / valuation | How is its stock valued? | Valuation class and price-related data |

A Material can exist generally yet remain unavailable for a particular sale because it has not been extended to the required Distribution Chain or Plant.

## Customer-material information record

```text
Customer
  + Sales Organization
  + Distribution Channel
  + Material
  = Customer-Material Information
```

This record stores facts that are true only for a particular customer/material commercial relationship. Examples include:

- the customer's own material number,
- the customer's description,
- customer-specific delivery information, and
- agreed delivery tolerances.

It is more specific than the Material master because the Material master describes the company's reusable product, whereas the customer-material record describes how one particular customer identifies or handles that product.

If the customer-material record is removed, the internal Material can still exist and be ordered by its internal number. What disappears is the customer-specific mapping and related convenience—for example, resolving customer number `BW-900` to internal Material `M-BIKE-01`.

## Worked source trace

| Sales Order field | Likely source | Operation | Scope |
|---|---|---|---|
| Sold-to Party | User input or reference document | Entered or copied | Header |
| Ship-to Party | Sold-to partner relationships | Proposed | Sales Area; usually header |
| Payment Terms | Customer data | Proposed | Relevant customer organizational scope |
| Internal Material | Customer-material record | Resolved from customer material number | Customer + Distribution Chain + Material |
| Sales Unit | Material or more-specific customer-material data | Proposed | Field-specific |
| Delivering Plant | Standard Plant determination using master and document inputs | Determined | Item |
| Shipping Point | Plant + loading group + shipping condition + Customizing | Derived | Shipping context |

Always verify the actual document type, configuration, current field values, organizational extensions, and release-specific logic when diagnosing a real system.

## RAP design transfer

```text
SalesOrderRequest
  ├── composition → Items
  ├── composition → OrderPartners
  │     └── association → BusinessPartner
  ├── association → SoldToBusinessPartner
  ├── association → SalesArea
  └── association → DeliveringPlant

Item
  ├── association → Product
  └── optional association → CustomerProductMapping
```

The reusable Business Partner exists independently, so it is referenced through an association. An `OrderPartner` row represents the transaction-specific fact that a particular BP performs a function in this order, so it may be a composition child of the order.

```text
BusinessPartner C110                = reusable master object
C110 is Ship-to for Order 4711      = transactional partner assignment
```

Decide deliberately whether an order stores a copied snapshot or only dereferences the current master record. If the UI always reads the current BP address through an association, historical orders may appear to change when the master address changes.

## Most important insights

### 1. “Automatic” is not a source

When SAP fills a field, identify the source object, organizational scope, operation, and determination logic. This is the level of precision needed for debugging and interviews.

### 2. A default becomes transactional data after acceptance

Master data may propose payment terms or partners, but the accepted values belong to the Sales Order. A permitted manual override changes the order, not the source master record.

### 3. More-specific data describes a relationship

Global Material data describes a product. Sales views describe how it is sold. A customer-material record describes the relationship between one customer and that product. The narrower context can provide more-specific values.

### 4. Existence is not usability

A BP or Material may exist generally but remain unusable for a particular transaction because the necessary Company Code, Sales Area, Distribution Chain, or Plant-specific data is missing.

### 5. Partner identity and partner responsibility are separate

The BP is a reusable party. The partner function is the responsibility assigned to that party in a process or document. This distinction maps naturally to association versus composition in RAP.

### 6. Historical truth requires an explicit snapshot decision

Referencing live master data and storing copied transaction values have different historical behavior. The data model must make that choice intentionally, especially for addresses and other values that can change later.

## Corrections and refinements from the exercise

| Initial idea | More precise understanding |
|---|---|
| Master data is simply a reusable “fact.” | It is reusable, independently maintained data with an organizational scope and lifecycle. |
| Sales Order data has meaning only for one sale. | Correct; it is also a transactional snapshot assembled from entered, copied, proposed, derived, or overridden values. |
| BP Category means Person or Group. | Include **Organization** as the third standard category. |
| BP Role is simply where a BP operates. | It enables an application context and the data segments required for that context. |
| General BP data includes identity, Company Code data includes payment, and Sales Area data includes sales information. | Correct at a high level; Company Code focuses on FI/AR, while Sales Area controls the customer relationship for sales, shipping, and billing. |
| Material views exist because some records matter only in particular situations. | Correct; the views also assign data to the organizational level and business function that owns it. |
| A customer-material record maps to relevant master data. | More precisely, it stores customer/product relationship facts and can map the customer's material number to the internal Material. |
| `OrderPartners` must be composition because an order is affected without them. | Composition is appropriate because each row records an order-specific assignment and shares the order lifecycle—not merely because the order needs partners. |
| A new order receives updated master data. | Correct; an existing order normally keeps previously copied values. Address handling is a notable exception. |

## Polished answers to all ten retrieval questions

### 1. How does master data differ from transactional Sales Order data?

Master data contains reusable information about Business Partners, Materials, and customer/material relationships. It has an independent lifecycle, is maintained for defined organizational scopes, and can support many transactions.

Sales Order data records the values accepted for one specific commercial transaction. Those values may be entered, copied, proposed, derived, or overridden. Once accepted, they normally form a historical transaction snapshot and are not automatically replaced when the source master data later changes.

### 2. What major source classes can provide or determine Sales Order values?

The major sources are reference documents, master data, Customizing, values already present in the Sales Order, and user input. Runtime determination logic evaluates these sources in a field-specific sequence.

For example, a quotation can supply copied values, customer and Material records can propose defaults, Customizing can define determination rules, existing fields can become inputs to later determinations, and users can enter or override permitted values.

### 3. How do BP Category, BP Role, and Partner Function differ?

A **BP Category** defines the fundamental kind of party: Person, Organization, or Group.

A **BP Role** enables the Business Partner to operate in an application context and provides the required data segments, such as Customer, FI Customer, or Supplier.

A **Partner Function** describes the responsibility that a partner performs in a particular sales process or document, such as Sold-to, Ship-to, Bill-to, or Payer.

### 4. What is the purpose of general, Company Code, and Sales Area BP data?

**General data** stores identity information shared across application contexts, such as name, address, and contact information.

**Company Code data** supports Accounts Receivable and financial processing for one accounting unit, including reconciliation-account, payment, and correspondence information.

**Sales Area data** defines the customer relationship for one Sales Organization, Distribution Channel, and Division, including sales, shipping, billing, partner, and commercial defaults.

### 5. What responsibilities belong to Sold-to, Ship-to, Bill-to, and Payer?

The **Sold-to Party** places or owns the commercial order relationship. The **Ship-to Party** receives the goods. The **Bill-to Party** receives the invoice. The **Payer** is responsible for settling the receivable.

These functions describe responsibilities in the transaction, not four mandatory distinct Business Partners.

### 6. Can one Business Partner perform all four functions, and can different partners perform them?

Yes. One Business Partner can perform Sold-to, Ship-to, Bill-to, and Payer for a simple customer relationship. In a more complex relationship, different Business Partners can perform the functions—for example, headquarters orders, a warehouse receives the goods, an accounts-payable office receives the invoice, and a treasury entity pays.

Partner determination and Customizing control which partner functions are allowed, required, proposed, or changeable.

### 7. Why does Material master data use different views and organizational scopes?

One Material is used by multiple business functions and locations. Separate views allow each function to maintain the data it needs at the correct organizational scope without duplicating the product as unrelated records.

General data identifies the product, sales data controls how it is sold through a Distribution Chain, Plant data supports planning and fulfillment at a location, and accounting data controls valuation. A Material must be extended to the scopes required by the intended transaction.

### 8. What does a customer-material information record represent, and why is it more specific than the Material master?

A customer-material information record represents the commercial relationship between a particular customer and a particular Material for a Sales Organization and Distribution Channel. It can store the customer's own material number and description, delivery information, and agreed tolerances.

It is more specific because the Material master describes the company's reusable product, while the customer-material record describes facts that apply only when that customer buys that product.

### 9. What normally happens to an existing Sales Order after non-address customer master data changes?

The existing Sales Order normally retains the values that were previously copied or proposed and accepted. For example, changing customer payment terms does not normally replace the payment terms already stored in an existing order. A newly created order can use the updated master data.

Customer address handling is a notable exception, including time-dependent address behavior. Its exact effect should be checked against the configured SAP process rather than treated like an ordinary copied field.

### 10. How should reusable Business Partners and order-specific partner assignments be modeled differently in RAP?

Reusable Business Partners should remain independent entities referenced by associations because they exist outside any single order and can be reused by many transactions.

Order-specific partner assignments can be modeled as composition children of the Sales Order root because each assignment records a fact belonging to that order—for example, “BP C110 is the Ship-to Party for this request.” Each assignment then associates to the reusable Business Partner.

## Active-recall keywords

Cover the right column and reconstruct each explanation aloud.

| Cue | What to retrieve |
|---|---|
| Data provenance | Field → source → scope → operation → transaction → later change |
| Source classes | Reference, master, Customizing, existing values, user input |
| Transactional snapshot | Accepted order values normally remain historically stable |
| Category | Fundamental party type: Person, Organization, Group |
| Role | Application context and required BP data segments |
| Function | Responsibility in a transaction |
| General BP data | Shared identity and contact data |
| Company Code BP data | FI, AR, reconciliation, payment, correspondence |
| Sales Area BP data | Sales, shipping, billing, partners, defaults |
| Sold-to | Owns or places the commercial order |
| Ship-to | Receives the goods |
| Bill-to | Receives the invoice |
| Payer | Settles the receivable |
| Material views | One product, multiple functions and organizational scopes |
| Material extension | Existence generally does not guarantee transactional usability |
| Customer-material info | Customer-specific identity and handling of one Material |
| Updated master data | New order uses it; existing order normally keeps its snapshot |
| Address exception | Specialized and potentially time-dependent behavior |
| BP association | Reusable independent master object |
| OrderPartner composition | Transaction-specific assignment owned by the order |
| Proposed ≠ master changed | Override affects the transaction, not the master record |

## Ten-second recall

```text
Master data   = reusable source
Sales Order   = transactional snapshot
Category      = what the party is
Role          = where the party operates
Function      = what the party does
BP            = independent association
OrderPartner  = order-owned composition
New order     = latest applicable defaults
Existing order = normally keeps accepted values
```

## Primary review sources

- [SAP Learning — Identifying the Source of Data in a Sales Document](https://learning.sap.com/courses/fundamental-customizing-in-sap-s-4hana-sales/identifying-the-source-of-data-in-a-sales-document)
- [SAP Learning — Maintaining Business Partner Master Data](https://learning.sap.com/courses/exploring-sap-s-4hana-sales-essentials/maintaining-business-partner-master-data_ce0c428f-2cad-4126-9f9a-1a263c3cc41b)
- [SAP Learning — Maintaining Material Master Data](https://learning.sap.com/courses/exploring-sap-s-4hana-sales-essentials/maintaining-material-master-data_db87a1fe-aa83-456b-9540-fd9561195213)
- [SAP Learning — Applying the Partner Function Concept](https://learning.sap.com/courses/fundamental-customizing-in-sap-s-4hana-sales/applying-the-partner-function-concept)
- [SAP Help — Customer Material Information](https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/7b24a64d9d0941bda1afa753263d9e39/3c8bc95360267214e10000000a174cb4.html)

## Suggested spaced recall

- **Tomorrow:** Reconstruct the three BP concepts and three BP data levels without notes.
- **In three days:** Trace Payment Terms, Ship-to, Material, Plant, and Shipping Point using the six-step answer frame.
- **In one week:** Answer all ten questions aloud, then draw the RAP association/composition model from memory.

