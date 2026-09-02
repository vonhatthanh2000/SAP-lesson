# Lesson 12 Recap: SAP SD Enterprise Structure

## Lesson objective

Given a proposed sale, identify:

1. which accounting entity records the result,
2. who is commercially responsible for selling,
3. how the product reaches the market,
4. which product or service group is sold,
5. from which operational location fulfillment can occur, and
6. which shipping unit processes the delivery.

The goal is to understand how organizational assignments control valid business paths—not merely memorize organizational-unit names.

## The lesson in one sentence

SAP SD enterprise structure is a control graph that defines valid commercial and fulfillment combinations; the existence of individual organizational units does not automatically permit them to work together.

## The six business questions

| Question | Organizational unit | Main responsibility |
|---|---|---|
| Which independent accounting unit records the result? | Company Code | Legal and financial reporting |
| Who sells? | Sales Organization | Commercial sales responsibility |
| How is it sold? | Distribution Channel | Route to market |
| What product family is sold? | Division | Product or service grouping |
| From where is it fulfilled? | Plant | Operational production, distribution, or service location |
| Which unit executes shipping? | Shipping Point | Delivery and shipping execution |

## The two formulas

### Sales Area

```text
Sales Area
= Sales Organization
+ Distribution Channel
+ Division
```

Memory sentence:

```text
Sales Area = WHO sells + HOW it is sold + WHAT is sold
```

Example:

```text
V100 / 20 / SP
Vietnam Domestic Sales / Wholesale / Spare Parts
```

The Sales Area is the complete commercial combination. The exact combination must be configured as valid.

### Distribution Chain

```text
Distribution Chain
= Sales Organization
+ Distribution Channel
```

Memory sentence:

```text
Distribution Chain = WHO sells + HOW it is sold
```

The Distribution Chain does not include the Division. Delivering Plants are assigned to this Sales Organization and Distribution Channel combination.

## Commercial, fulfillment, and execution layers

```text
Financial context
  Company Code
       ↑
Commercial context
  Sales Organization + Distribution Channel + Division
       ↓
Fulfillment permission
  Distribution Chain → Plant
       ↓
Shipping execution
  Plant → Shipping Point
```

This is not a strict parent-child tree. It is a set of assignments that answers different business questions.

## Important assignment rules

| Assignment | What it permits or determines |
|---|---|
| Sales Organization → Company Code | Determines the accounting entity to which that selling unit belongs |
| Distribution Channel ↔ Sales Organization | Permits the seller to use that route to market |
| Division ↔ Sales Organization | Permits the seller to handle that product or service group |
| Sales Organization + Channel + Division | Creates a valid Sales Area combination |
| Distribution Chain ↔ Plant | Permits the Plant to fulfill for that seller and channel |
| Shipping Point ↔ Plant | Makes that shipping unit available for the Plant |

Key relationship:

```text
One Company Code → multiple Sales Organizations
One Sales Organization → exactly one Company Code
```

## The most important insights

### 1. Enterprise structure is more than an organizational chart

The units do not merely describe departments. Their assignments influence whether a transaction is organizationally valid and provide context for responsibility, master data, authorization, pricing, fulfillment, reporting, and financial integration.

When hearing a label such as “Vietnam Sales,” ask what it controls. It might refer to a Company Code, Sales Organization, Sales Office, or an informal business name.

### 2. Existing values do not guarantee a valid combination

These units may all exist:

```text
Sales Organization: V100
Channel:            10 Direct
Division:           SP Spare Parts
```

However, `V100/10/SP` is not a valid Sales Area unless that exact combination is configured.

```text
Valid individual values ≠ valid business combination
```

### 3. Commercial responsibility and physical fulfillment are separate

The Sales Area describes the commercial context: who sells, how, and what. A Plant describes the operational location from which fulfillment can occur.

This separation permits:

- one sales path to use multiple Plants,
- one Plant to serve multiple Distribution Chains, and
- configured cross-company fulfillment scenarios.

A Plant is therefore not part of the Sales Area formula.

### 4. An assignment gives permission, not a successful transaction

Assigning a Plant to a Distribution Chain means the Plant may serve as a delivering Plant for that Sales Organization and Channel.

It does not guarantee that a specific Sales Order can be delivered. The system may still require:

- a valid complete Sales Area,
- suitable Business Partner and material master data,
- stock and availability,
- Shipping Point determination,
- pricing,
- credit approval, and
- other process-specific controls.

```text
Allowed organizational path ≠ executable business transaction
```

### 5. Plant and Shipping Point answer different questions

A Plant is the operational unit that can manufacture, distribute, store relevant stock, or provide services. It answers: **from where can fulfillment occur?**

A Shipping Point controls shipping execution such as delivery processing, picking, packing, and dispatch. It answers: **which shipping unit executes the delivery?**

A Plant can have multiple Shipping Points. Each Outbound Delivery is processed by one Shipping Point.

### 6. Sales Office and Sales Group do not define the Sales Area

A Sales Office commonly represents a geographical market, branch, or sales responsibility area. A Sales Group represents a team or area of responsibility within a Sales Office.

They support responsibility assignment, analytics, reporting, and selection. They are not part of the core formula that defines who sells what through which channel.

```text
Sales Area includes:
  Sales Organization + Distribution Channel + Division

Sales Area does not include:
  Sales Office, Sales Group, Plant, Shipping Point
```

### 7. Organizational references are not RAP composition children

In the custom RAP model:

```text
SalesOrderRequest
  ├── composition → Items
  ├── association → Customer / Business Partner
  ├── association → Sales Area
  └── association → Delivering Plant
```

Items depend on the Request for their lifecycle, so composition is appropriate. Customer, Sales Area, and Plant exist independently and may be reused by many Requests, so they should be associations.

## Worked example: Velora Vietnam

### Organizational units

| Unit | Code | Meaning |
|---|---|---|
| Company Code | `VN01` | Velora Vietnam accounting entity |
| Sales Organization | `V100` | Vietnam Domestic Sales |
| Direct Channel | `10` | Direct customer sales |
| Wholesale Channel | `20` | Wholesale sales |
| Bicycle Division | `BI` | Bicycles |
| Spare Parts Division | `SP` | Spare parts |
| Plants | `HCM1`, `HAN1` | Ho Chi Minh and Hanoi fulfillment Plants |
| Shipping Points | `HCMP`, `HCMT`, `HANT` | Parcel and truck shipping units |

### Configured Sales Areas

```text
V100 / 10 / BI  → direct bicycle sales
V100 / 20 / BI  → wholesale bicycle sales
V100 / 20 / SP  → wholesale spare-parts sales
```

Missing:

```text
V100 / 10 / SP  → direct spare-parts sales are not configured
```

### Fulfillment assignments

```text
V100 / 10 → HCM1
V100 / 20 → HCM1
V100 / 20 → HAN1
```

The first three values describe commercial permission. The Plant assignments describe allowed fulfillment paths. They are related but separate configuration facts.

## Corrections to remember

| Initial idea | More precise understanding |
|---|---|
| Company Code is the highest level controlling the whole system. | Company Code is an independent accounting unit; it is not the highest technical level and does not define the legal context for the whole system. |
| Distribution Channel describes how the product is physically transferred. | It describes the route to market, such as direct, retail, or wholesale; physical shipping belongs to fulfillment. |
| Division means an individual product or item. | Division groups a product or service family, such as bicycles or spare parts. |
| Distribution Chain focuses only on the route to market. | It combines the seller and route: Sales Organization + Distribution Channel. |
| Plant is excluded because fulfillment could be outsourced. | Plant is separate because it represents operational fulfillment, while Sales Area represents commercial responsibility. Outsourcing is only one possible scenario. |
| Plant assignment guarantees shipping if the Shipping Point works. | It only permits a fulfillment route; master data, stock, Shipping Point determination, pricing, credit, and other checks still matter. |
| A Sales Area fails because one component contains a broken condition. | The exact combination itself may be missing even though every individual value is valid. |
| Sales Office is always a physical building. | It can represent a regional, geographical, branch, or responsibility area. |
| Sales Office and Group are excluded mainly to support scaling. | They are excluded because they organize responsibility and reporting rather than define the who/how/what commercial combination. |

## Polished answers to all ten questions

### 1. How do Company Code and Sales Organization differ?

A **Company Code** is an independent accounting unit for which complete financial statements can be produced. Its focus is legal and financial reporting.

A **Sales Organization** is the central SD unit responsible for selling and distributing goods or services, negotiating sales conditions, and commercial responsibility.

One Company Code can contain multiple Sales Organizations, but each Sales Organization is assigned to exactly one Company Code. The assignment determines which accounting entity receives the financial results of that Sales Organization's transactions.

### 2. Which units form a Sales Area, and what does each contribute?

A Sales Area consists of:

- **Sales Organization — who sells:** the unit commercially responsible for the sale.
- **Distribution Channel — how it is sold:** the route to market, such as direct, retail, or wholesale.
- **Division — what category is sold:** a product or service group, such as bicycles, spare parts, or services.

Together, they define who may sell what through which channel.

### 3. How do Sales Area and Distribution Chain differ?

A **Sales Area** combines Sales Organization, Distribution Channel, and Division. It represents who sells what through which route to market.

A **Distribution Chain** combines only Sales Organization and Distribution Channel. It represents who sells through which channel without identifying the Division. Delivering Plants are assigned to Distribution Chains.

### 4. Why is Plant not part of the Sales Area formula?

The Sales Area defines the commercial context: who sells, through which channel, and which product group is sold.

A Plant represents the operational location that manufactures, distributes, or provides the product or service. It answers where fulfillment can occur rather than who is commercially responsible for the sale. It is therefore assigned separately to a Distribution Chain.

### 5. What does assigning a Plant to a Distribution Chain permit?

The assignment permits that Plant to be used as a delivering Plant for the specified Sales Organization and Distribution Channel.

It does not guarantee that a particular Sales Order can be delivered. The full Sales Area and relevant master data must be valid, stock may need to be available, a Shipping Point must be determined, and pricing, credit, and other process checks may still affect execution.

### 6. How do Plant and Shipping Point differ?

A **Plant** is an operational unit that can manufacture products, distribute goods, manage relevant stock, or provide services. It represents the location from which fulfillment can occur.

A **Shipping Point** is the unit responsible for shipping execution, including delivery processing, picking, packing, and dispatch. Shipping Points are assigned to Plants, and each Outbound Delivery is processed by one Shipping Point.

### 7. Why can existing units still fail to form a valid Sales Area?

A Sales Organization, Distribution Channel, and Division can each exist independently without being permitted to operate together. The exact combination must be configured as a Sales Area.

For example, `V100`, channel `10`, and Division `SP` can all exist, but direct spare-parts sales are unavailable if `V100/10/SP` has not been configured.

### 8. Where do Sales Office and Sales Group fit?

A **Sales Office** represents a regional, geographical, branch, or organizational sales responsibility. A **Sales Group** represents a team or area of responsibility within a Sales Office.

They support responsibility assignment, reporting, analytics, and selection. They are excluded from the Sales Area formula because they do not define the core commercial combination of who sells, how, and what.

### 9. What enterprise-structure fact is missing for `V100/10/SP`?

The exact `V100/10/SP` Sales Area has not been configured. Although the individual Sales Organization, Direct Channel, and Spare Parts Division exist, there is no configuration permitting V100 to sell spare parts through the Direct channel.

### 10. Which RAP relationships should be composition or association?

`Item` should be a **composition child** of `SalesOrderRequest` because it is lifecycle-dependent on the Request and should not exist independently after its parent is deleted.

`Customer`, `Sales Area`, and `Plant` should be represented by **associations** because they are independently meaningful master or configuration objects. They exist before the Request, can be referenced by many Requests, and are not deleted with it.

## Active-recall keywords

Hide the right column and reconstruct the meaning aloud.

| Cue | What to retrieve |
|---|---|
| Company Code | Independent accounting and legal reporting unit |
| Sales Organization | Commercial seller; one Company Code |
| Distribution Channel | Route to market: direct, retail, wholesale |
| Division | Product or service family |
| Sales Area | Sales Org + Channel + Division |
| Who/how/what | Sales Area memory formula |
| Distribution Chain | Sales Org + Channel |
| Who/how | Distribution Chain memory formula |
| Plant | Operational fulfillment location |
| Shipping Point | Shipping execution unit |
| Sales Office | Regional or organizational sales responsibility |
| Sales Group | Team or responsibility area within an office |
| Exact combination | Existing units do not automatically form a Sales Area |
| Plant assignment | Permitted delivery source, not guaranteed fulfillment |
| Association | Independent reusable reference |
| Composition | Dependent child lifecycle |
| V100/10/SP | Missing configured Sales Area |

## Ten-second contrast list

```text
Company Code       = financial responsibility
Sales Organization = commercial responsibility

Distribution Channel = route to market
Shipping Point       = physical shipping execution

Sales Area        = who + how + what
Distribution Chain = who + how

Plant assignment  = permitted fulfillment
Successful order  = all required controls pass

Existing units    ≠ valid combination
Association       ≠ composition ownership
```

## Ninety-second interview answer

> SAP SD enterprise structure defines the organizational context in which a sale can be processed. The Company Code is the independent accounting unit, while the Sales Organization is commercially responsible for selling. A Sales Area combines the Sales Organization, Distribution Channel, and Division, meaning who sells what through which route to market. A Distribution Chain contains only the Sales Organization and Distribution Channel. Delivering Plants are assigned to Distribution Chains, while Shipping Points are assigned to Plants and execute shipping activities. The existence of the individual units is insufficient—the exact Sales Area and fulfillment assignments must be configured. Even then, the assignments only permit the organizational path; master data, stock, pricing, credit, and other checks still determine whether a particular order can be executed.

## Spaced active-recall plan

- **Tomorrow:** reproduce both formulas and define the six core units without notes.
- **After 3 days:** diagnose why `V100/10/SP` fails and explain why a Plant assignment does not guarantee delivery.
- **After 7 days:** draw the complete commercial-to-shipping control map and answer the questions in random order.
- **After 14 days:** deliver the ninety-second answer and connect every organizational reference to your RAP model.

## Sources

- [Lesson 12 — Map the SAP SD Enterprise Structure](../lessons/0012-map-sd-enterprise-structure.html)
- [SAP Learning — Identifying Organizational Units in SAP S/4HANA Sales](https://learning.sap.com/courses/exploring-sap-s-4hana-sales-essentials/identifying-organizational-units-in-sap-s-4hana-sales_ddb48d1e-1f57-46bd-8d13-d8e4ef6d0560)
- [SAP Learning — Setting Up the Enterprise Structure in Sales and Distribution](https://learning.sap.com/courses/fundamental-customizing-in-sap-s-4hana-sales/setting-up-the-enterprise-structure-in-sales-and-distribution)
- [Complete SAP SD Source Map](../reference/sap-sd-source-map.html)
