# Lesson 14 Recap: Sales Order Header, Item, and Schedule-Line Levels

## Lesson objective

Given any Sales Order field or processing problem, determine:

1. whether it belongs to the header, item, or schedule-line level,
2. the business scope represented by that level,
3. how values propagate downward or aggregate upward,
4. the correct database and API grain, and
5. which data and configuration inputs to inspect during diagnosis.

The goal is to reason from business scope and data grain—not from where a field happens to appear on a screen.

## The lesson in one sentence

A Sales Order header represents one commercial transaction, each item represents one product or service line, and each schedule line represents one dated quantity commitment within an item.

## The central model

```text
Sales Order header                one commercial transaction
├── Item 10                       one product/service line
│   ├── Schedule line 0001        one quantity/date commitment
│   └── Schedule line 0002        another quantity/date commitment
└── Item 20
    └── Schedule line 0001
```

The placement rule is:

> Place a value at the broadest level where it remains true without losing business meaning.

Ask:

```text
Does it apply to the whole order?        → Header
Does it apply to one product/service?    → Item
Does it apply to one quantity and date?  → Schedule line
```

## The three business scopes

| Level         | Business question                                                                              | Typical fields                                                                                                                  |
| ------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Header        | Who is this transaction with, and under which common commercial context?                       | Sales Order number/type, Sales Area, Sold-to Party, customer reference, document currency, total value, overall status or block |
| Item          | What product or service is being sold, in what total quantity, and how is this line processed? | Material, ordered quantity, sales unit, Plant, item category, unit price, net amount, rejection reason                          |
| Schedule line | How much is requested or confirmed, and for which date?                                        | Requested quantity/date, confirmed quantity/date, delivered quantity, open confirmed quantity, schedule-line block              |

Screen position does not determine the data level. A value displayed near the top of a UI is not automatically a header field, and a visually repeated value may be stored separately for each item.

## Requested, confirmed, delivered, and open quantities

| Quantity           | Meaning                                                                                                                      |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| Ordered quantity   | Total quantity requested for one Sales Order item                                                                            |
| Requested quantity | Portion of demand requested for a particular delivery date                                                                   |
| Confirmed quantity | Quantity SAP currently commits to fulfilling on a specified date after scheduling and availability checking                  |
| Delivered quantity | Quantity already processed into delivery; verify PGI separately to prove physical goods issue                                |
| Open quantity      | Quantity that remains eligible for further processing; distinguish open order quantity from open confirmed delivery quantity |

Memory pattern:

```text
Requested = customer demand
Confirmed = company commitment
Delivered = fulfillment progress
Open      = remaining eligible quantity
```

Important boundaries:

```text
Confirmed quantity ≠ delivered quantity
Delivery created   ≠ PGI posted
```

## Partial-confirmation example

BikeWorld orders 10 bicycles for 10 September. Six can be confirmed for that date and four for 17 September.

```text
Item 10
  Ordered quantity:   10
  Confirmed total:    10

  Schedule line 0001
    Confirmed quantity: 6
    Confirmed date:     10 Sep

  Schedule line 0002
    Confirmed quantity: 4
    Confirmed date:     17 Sep
```

This item is fully confirmed across two dates, but the scenario does not say that anything has been delivered or PGI-posted. Schedule lines describe fulfillment commitments; an Outbound Delivery executes due, delivery-relevant commitments.

The exact schedule-line representation can depend on configuration and the availability result. Preserve the invariant:

```text
Item          = overall product demand
Schedule line = dated quantity commitment
```

## Header defaults and item exceptions

Selected header values may be copied or propagated into corresponding item fields. This is not the same as every item dynamically reading one shared header field.

```text
Initial state
Header payment terms: 30 days
Item 10:              30 days  ← aligned with header
Item 20:              45 days  ← explicit exception

Header changes to 60 days
Item 10:              60 days  ← may follow header propagation
Item 20:              45 days  ← may retain its exception
```

The exact result is field-specific and may also depend on document status and follow-up documents.

Use these words precisely:

```text
Copied/propagated = a value is transferred under defined rules
Dynamic reference = the child always reads one shared parent value
Item exception    = the item contains a permitted different value
```

## Values and statuses also roll upward

Information does not move only from header to item. Child results contribute to parent summaries:

```text
Schedule-line confirmations → item confirmation/delivery state
Item values                 → header total value
Item statuses               → overall document status
```

An overall header status is therefore a derived summary. If one item is complete and another is open, the order can be partially processed. To find the cause, move downward from the header summary to the item and schedule-line states.

## Technical Consultant view: transactions, tables, keys, and API nodes

### Classic SAP GUI navigation

| Purpose             | Transaction |
| ------------------- | ----------- |
| Create Sales Order  | `VA01`      |
| Change Sales Order  | `VA02`      |
| Display Sales Order | `VA03`      |
| List Sales Orders   | `VA05`      |

These are classic SAP GUI landmarks. The available entry point may instead be a Fiori app or API depending on the S/4HANA edition and system design.

### Classic persistence grain

| Level         | Table landmark | Business key, ignoring client | One row represents               |
| ------------- | -------------- | ----------------------------- | -------------------------------- |
| Header        | `VBAK`         | `VBELN`                       | One sales document header        |
| Item          | `VBAP`         | `VBELN + POSNR`               | One item within a sales document |
| Schedule line | `VBEP`         | `VBELN + POSNR + ETENR`       | One schedule line within an item |

The keys are locally scoped:

```text
VBELN                         → identifies the sales document
VBELN + POSNR                 → identifies an item inside that document
VBELN + POSNR + ETENR         → identifies a schedule line inside that item
```

`ETENR` identifies the schedule-line record. The record contains dated quantities, but `ETENR` itself is not a quantity or date.

### Join path

```text
VBAK-VBELN
    =
VBAP-VBELN

VBAP-VBELN + VBAP-POSNR
    =
VBEP-VBELN + VBEP-POSNR
```

Use the full key required by the target grain. Querying `VBEP` by `VBELN` alone returns schedule lines from every item in that document.

### API hierarchy

For the SAP Sales Order A2X API family, the hierarchy is commonly exposed through technical entity names such as:

```text
A_SalesOrder
└── A_SalesOrderItem
    └── A_SalesOrderScheduleLine
```

The `A_` prefix is an SAP naming convention used by particular delivered API entities. It is not an ABAP keyword and is not required for every CDS entity or OData service.

```text
I_...           commonly identifies an interface CDS entity
C_...           commonly identifies a consumption/query entity
A_...           used by API entities in some SAP-delivered APIs
ZI_... / ZC_... common customer-namespace conventions
```

Always inspect the service definition or OData `$metadata` and use the exact exposed entity name. Another service version, including a V4 service, may expose different names.

### Tables are landmarks, not integration contracts

`VBAK`, `VBAP`, and `VBEP` help an authorized technical consultant understand persistence in applicable classic systems. They should not be updated directly, and a clean-core extension should not assume raw tables are a stable public contract.

Prefer released CDS entities, business objects, events, or APIs supported in the target system. They carry business semantics, authorization, lifecycle behavior, and compatibility commitments that raw tables do not.

## The wrong-grain aggregation trap

Suppose one header stores a total of `1,000`, the order contains two items, and each item contains two schedule lines.

```text
VBAK → 2 VBAP rows → 4 VBEP rows
```

After joining all levels:

| Order  | Item | Schedule line | Repeated header total |
| ------ | ---- | ------------- | --------------------: |
| 500001 | 0010 | 0001          |                 1,000 |
| 500001 | 0010 | 0002          |                 1,000 |
| 500001 | 0020 | 0001          |                 1,000 |
| 500001 | 0020 | 0002          |                 1,000 |

`SUM(HeaderTotal)` now returns `4,000`, although the actual header total is `1,000`.

The problem is not item override behavior. It is row multiplication caused by joining a parent to multiple children.

> Aggregate a measure at the grain where that measure is defined.

Safe conceptual strategies include:

- reading one header value per `VBELN`,
- aggregating schedule-line data to item grain before joining it to items,
- aggregating item data to header grain before combining it with header measures, and
- keeping quantity units and currencies in the grouping and conversion design.

Do not use `SUM(DISTINCT HeaderTotal)` as a general fix: two different orders may legitimately have the same total, and `DISTINCT` would then remove valid values.

## Confirmed-quantity diagnostic flow

When a user reports that the confirmed quantity for Item 10 is wrong:

```text
Identify exact item
      ↓
Inspect item demand
      ↓
Inspect every schedule-line result
      ↓
Reconstruct determination inputs
      ↓
Classify root cause
```

### 1. Identify the exact item

Use `VBELN + POSNR`. Do not diagnose only from a product description or UI row position.

### 2. Inspect item-level demand

Check:

- ordered quantity and unit,
- Material,
- delivering Plant,
- requested delivery date,
- rejection reason,
- delivery or other relevant blocks, and
- item processing state.

### 3. Inspect schedule-line results

Read every line using `VBELN + POSNR + ETENR` and compare:

- requested quantity and date,
- confirmed quantity and date,
- delivered quantity,
- open confirmed quantity, and
- schedule-line blocks.

Do not combine quantities until their units and business meanings match.

### 4. Reconstruct determination inputs

Depending on the scenario and system, inspect:

- Material and Plant-specific master data,
- stock, receipts, and competing requirements,
- availability-check controls,
- item and schedule-line categories,
- scheduling times and calendars,
- delivery blocks,
- configuration and master-data validity, and
- custom enhancements or integration input.

### 5. Classify the root cause

Decide whether the observed result comes from:

- incorrect transaction input,
- missing or incorrect master data,
- current availability,
- configuration,
- scheduling logic,
- custom code, or
- an interface payload.

Observe the stored demand and result first. Do not change configuration before proving that it is wrong and understanding which other transactions would be affected.

## RAP design transfer

```text
SalesOrderRequest                     root / header
├── composition [0..*] → Items        product/service lines
│   ├── association → Product
│   ├── association → Plant
│   └── composition [0..*] → Schedules
├── association → Customer/BusinessPartner
├── association → SalesArea
└── composition [0..*] → OrderPartners
```

Possible field ownership:

```text
Request
  SalesArea, SoldTo, Currency, OverallStatus, TotalAmount

Item
  Product, OrderedQuantity, Unit, Plant, UnitPrice, NetAmount

Schedule
  RequestedDate, ConfirmedDate, RequestedQty, ConfirmedQty
```

Composition is appropriate for `Request → Items → Schedules` because each child shares the lifecycle of its parent. Product, Business Partner, Plant, and Sales Area remain associations because they exist independently and can be reused across many transactions.

In S/4HANA, Customer is normally represented through the Business Partner model. Avoid creating duplicate `Customer` and `BusinessPartner` master entities unless they are deliberately different projections or roles.

Your custom `Schedules` entity can represent a simplified promise plan, but it must not be presented as a reimplementation of standard SAP ATP, requirements transfer, scheduling, or delivery processing.

## Most important insights

### 1. Business scope determines field placement

Do not classify a field based on the UI or a memorized list. Ask where the statement remains true: whole order, one product line, or one dated quantity.

### 2. One product line can contain several commitments

An item expresses total product demand. Schedule lines express when portions of that demand are requested or confirmed.

### 3. Confirmation is a promise, not physical execution

A confirmed quantity can support later delivery creation, but it does not prove that a delivery exists or that PGI occurred.

### 4. Copying is not dynamic inheritance

Header values can be copied or propagated to items under field-specific rules. An item can store a permitted exception and may then stop following the common header value.

### 5. Overall status hides lower-level detail

Header status summarizes child states. Debug a partial or inconsistent status by drilling down through items and schedule lines.

### 6. Database grain is part of business correctness

A technically valid join can still produce a wrong business result. Parent measures repeat at child grain, so aggregation must respect the measure's defining level.

### 7. A table is not a business-object contract

Classic persistence knowledge helps diagnosis. Released APIs and CDS/business-object contracts are the safer boundary for extensions and integrations.

### 8. Diagnosis should reconstruct causality

Start from the exact transaction and stored result, then reconstruct the inputs that produced it. Do not begin by changing code or configuration.

## Corrections and refinements from the exercise

| Initial idea                                                                 | More precise understanding                                                                                                                         |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Schedule-line values belong there because they relate to the delivery stage. | Schedule lines already exist inside the Sales Order and represent dated fulfillment commitments before an Outbound Delivery is created.            |
| Requested quantity is what exists or what the Sales Organization prepares.   | Requested quantity expresses customer demand for a date; availability influences the confirmed quantity.                                           |
| Confirmed quantity is the quantity expected to be delivered.                 | It is the quantity SAP currently commits for a date; it does not prove delivery execution.                                                         |
| Open quantity is simply stock stored for future delivery.                    | It is remaining quantity eligible for processing; the exact open measure and rejection/cancellation status matter.                                 |
| Items reference one header value, except for special properties.             | Selected values are copied or propagated into item fields; each item can hold its own value and exceptions.                                        |
| Header status contains the total item or fully delivered date.               | Header status is an aggregate summary derived from lower-level processing states; dates and quantities are supporting data, not the status itself. |
| Six confirmed now and four later means quantities were delivered.            | It proves full confirmation across two dates, not delivery creation or PGI.                                                                        |
| `ETENR` identifies a date and quantity.                                      | `ETENR` identifies a schedule-line record that contains date and quantity data.                                                                    |
| Header totals inflate because item values may override them.                 | They inflate because one-to-many joins repeat the same parent value at child-row grain.                                                            |
| Joining the three tables is enough to diagnose confirmation.                 | Begin with the exact keys, inspect demand and each schedule result separately, then reconstruct determination inputs.                              |

## Polished answers to all ten retrieval questions

### 1. What business question does each level answer?

The **header** contains information applying to the complete Sales Order, such as the Sold-to Party, Sales Area, document currency, total, and overall status.

An **item** represents one product or service line. It contains the Material, total ordered quantity, unit, Plant, price, net amount, and line-specific controls.

A **schedule line** divides an item's requirement by quantity and date. It stores requested and confirmed quantities and delivery dates, along with fulfillment-related progress and controls.

### 2. Why is Material normally an item-level value?

Material identifies the particular product or service sold on one line. A Sales Order can contain several items with different Materials, quantities, Plants, prices, and delivery requirements. A header-level Material would incorrectly imply that the complete order contains only one product.

### 3. Why are confirmed quantity and date schedule-line data?

One item's ordered quantity can be fulfilled in different portions on different dates. The item stores total demand, while every schedule line stores one dated fulfillment commitment produced through scheduling and availability checking.

### 4. Can one item have multiple schedule lines, and why?

Yes. Multiple schedule lines are needed when an item's total quantity is requested or confirmed in separate portions or on different dates. This can result from partial availability, later receipts, scheduling results, or deliberately staged customer requirements.

### 5. How do ordered, requested, confirmed, delivered, and open quantities differ?

Ordered quantity is the total demand for one item. Requested quantity associates demand with a requested date. Confirmed quantity is the quantity SAP currently commits for a specified date. Delivered quantity represents delivery-processing progress, while PGI must be checked separately for physical issue. Open quantity is the remaining eligible quantity, interpreted according to whether the metric is open order quantity or open confirmed delivery quantity.

### 6. Does a copied header value mean items dynamically reference one field?

No. SAP normally copies or propagates selected header values into corresponding item fields. Each item can therefore contain its own stored value; it does not merely read one shared header value dynamically.

### 7. What happens when the header changes after an item exception exists?

Items still aligned with the previous header value can receive the new header value. An item containing an explicitly different permitted value may retain that exception. The exact behavior is field-specific and can be restricted by document state or subsequent processing.

### 8. How do lower-level states influence header status?

Schedule-line confirmations and delivery progress contribute to item status, while item statuses contribute to the overall document status. The header status is an aggregate summary: if some items are complete and others remain open, the order can be partially processed. Diagnosis therefore requires drilling down to the contributing item and schedule-line states.

### 9. What is the state when 10 units are ordered, six confirmed now, and four later?

The item contains an ordered quantity of 10 and is fully confirmed in total. One schedule line confirms six units for the earlier date, and another confirms four units for the later date. The scenario does not establish that either quantity has been delivered or PGI-posted.

### 10. How should the hierarchy be modeled in RAP?

`SalesOrderRequest` should be the root/header and own its `Items` through composition. Each Item should own its `Schedules` through composition because a schedule has no independent lifecycle outside its item. Product, Business Partner, Plant, and Sales Area should remain associations because they are independently maintained and reused by many orders.

## Polished answers to all five technical checkpoints

### T1. What do `VBELN`, `POSNR`, and `ETENR` identify?

`VBELN` identifies the sales document. `POSNR` identifies an item within that document. `ETENR` identifies a schedule line within that item. Therefore, the business key for a schedule line is `VBELN + POSNR + ETENR`, ignoring the client field.

### T2. Which classic tables represent each grain?

`VBAK` contains one row per Sales document header at `VBELN` grain. `VBAP` contains one row per document item at `VBELN + POSNR` grain. `VBEP` contains one row per item schedule line at `VBELN + POSNR + ETENR` grain.

### T3. Why can joining the tables inflate a header total?

Joining a header to multiple items and schedule lines repeats the same header columns for every matching child row. Summing a header-level amount after the join therefore counts the same amount multiple times. Aggregate each measure at the grain where it is defined.

### T4. Which API nodes represent the same hierarchy?

In the Sales Order A2X API family, the corresponding technical entities are commonly `A_SalesOrder`, `A_SalesOrderItem`, and `A_SalesOrderScheduleLine`. They expose business semantics through a supported service contract rather than requiring consumers to depend directly on raw table layout. The exact names must always be verified in the service's `$metadata`.

### T5. How should an incorrect confirmed quantity be diagnosed?

First identify the item using `VBELN + POSNR`. Inspect item demand such as ordered quantity and unit, Material, Plant, requested date, and blocks. Then inspect each schedule line by the full `VBELN + POSNR + ETENR` key, comparing requested and confirmed quantities and dates as well as delivered and open quantities. Finally, reconstruct availability, master-data, scheduling, category, configuration, and custom-code inputs to classify the root cause.

## Active-recall keywords

Cover the right column and reconstruct each answer aloud.

| Cue                    | What to retrieve                                       |
| ---------------------- | ------------------------------------------------------ |
| Broadest true scope    | Rule for choosing header, item, or schedule line       |
| Header                 | Complete commercial transaction                        |
| Item                   | One product/service line and total demand              |
| Schedule line          | One quantity/date commitment                           |
| Ordered                | Total item demand                                      |
| Requested              | Customer demand for a date                             |
| Confirmed              | Company's current dated commitment                     |
| Delivered              | Delivery progress; PGI checked separately              |
| Open                   | Remaining eligible processing quantity                 |
| Confirmed ≠ delivered  | Promise is not physical execution                      |
| Copy/propagation       | Transfer into item fields under defined rules          |
| Item exception         | Permitted different item-level value                   |
| Downward               | Header values may propagate to aligned items           |
| Upward                 | Child values and states contribute to parent summaries |
| `VBELN`                | Sales document identity                                |
| `POSNR`                | Item identity within a document                        |
| `ETENR`                | Schedule-line identity within an item                  |
| `VBAK`                 | Header grain                                           |
| `VBAP`                 | Item grain                                             |
| `VBEP`                 | Schedule-line grain                                    |
| Row multiplication     | Parent repeated once per matching child                |
| Grain-safe aggregation | Aggregate where the measure is defined                 |
| `A_` prefix            | Convention in particular SAP API entity names          |
| `$metadata`            | Authority for exact exposed OData names                |
| Composition            | Transactional lifecycle ownership                      |
| Association            | Independent reusable object                            |
| Diagnosis              | Demand → schedule result → inputs → root cause         |

## Ten-second recall

```text
Header        = whole order
Item          = one product and total demand
Schedule line = one dated quantity

Requested ≠ confirmed ≠ delivered ≠ PGI

VBAK: VBELN
VBAP: VBELN + POSNR
VBEP: VBELN + POSNR + ETENR

Parent-to-child join repeats parent values
Aggregate at the measure's own grain
```

## Primary review sources

- [SAP Learning — Executing Sales Order Management](https://learning.sap.com/courses/executing-basic-erp-processes-with-sap-s-4hana/executing-sales-order-management)
- [SAP Help — Relationship Between Header and Items](https://help.sap.com/docs/SAP_S4HANA_CLOUD/a376cd9ea00d476b96f18dea1247e6a5/5664b65334e6b54ce10000000a174cb4.html)
- [SAP Help — Sales Order Item Schedule Line](https://help.sap.com/docs/SAP_S4HANA_CLOUD/03c04db2a7434731b7fe21dca77440da/37df44581efca007e10000000a441470.html)
- [SAP Learning — Running an Available-to-Promise Check](https://learning.sap.com/courses/performing-the-availability-check/running-an-available-to-promise-atp-check-in-sap-s-4hana-sales_a2b4c5e3-1618-418d-a4f6-efe5ff43f7f1)
- [SAP Help — APIs for Sales](https://help.sap.com/docs/sap_s4hana_cloud/03c04db2a7434731b7fe21dca77440da/f67705a25e2b440f90a25faaffa5ffef.html)

## Suggested spaced recall

- **Tomorrow:** Reconstruct the three levels, five quantity meanings, and three classic table keys without notes.
- **In three days:** Explain the header/item propagation example and diagnose the inflated-total join.
- **In one week:** Answer all 15 questions aloud and trace a wrong confirmation from item demand to root cause.
