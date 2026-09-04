# Lesson 15 Recap: Sales Document Type, Item Category, and Schedule-Line Category

## Lesson objective

Given a Sales Order processing problem, determine:

1. which business level controls the behavior,
2. which inputs caused SAP to select the category,
3. whether the wrong category was selected or the selected category was configured incorrectly,
4. which classic transaction, table landmark, and stored result help trace it, and
5. when to widen the investigation beyond standard configuration.

The goal is to diagnose from business grain and runtime evidence—not to memorize a disconnected list of transactions.

## The lesson in one sentence

The sales document type defines the overall transaction context, the item category defines how one product or service line participates, and the schedule-line category defines how one dated quantity participates in fulfillment.

## The central control chain

```text
Header: Sales document type
  │
  ├─ combines with material and item context
  ▼
Item: Item category
  │
  ├─ combines with material/plant planning context
  ▼
Schedule line: Schedule-line category
  │
  └─ controls fulfillment participation
```

| Control level | Business question | Examples of behavior controlled |
| --- | --- | --- |
| Sales document type | What kind of commercial transaction is this? | Document category, defaults, blocks, checks, follow-on document proposals |
| Item category | How does this particular line participate? | Pricing, billing, business-data differences, text/material behavior, schedule-line permission |
| Schedule-line category | How does this dated quantity participate in logistics? | Delivery relevance, requirements transfer, ATP, movement type, procurement, blocks |

The categories cooperate, but none replaces the others. Header context alone cannot describe how every line and dated quantity should behave.

## The most important technical distinction: determination versus definition

```text
Determination/assignment → Which category is selected?
Category definition      → What does that selected category do?
```

This distinction gives you the first diagnostic branch:

```text
Wrong category selected
→ inspect determination inputs and assignment

Correct category selected, wrong behavior
→ inspect category definition and related process configuration
```

Do not jump directly to custom code. First prove whether standard determination selected the wrong result or whether the selected result has unsuitable control settings.

## Item-category determination

SAP proposes the item category from four inputs:

```text
Sales document type
+ Material item-category group
+ Item usage
+ Higher-level item category
────────────────────────────────
= Proposed item category
```

### What each input means

| Input | Meaning | Technical clue |
| --- | --- | --- |
| Sales document type | Overall transaction context, such as inquiry or standard order | Selected value in `VBAK-AUART` |
| Material item-category group | General sales-processing classification of the material | Commonly `MVKE-MTPOS`; use the correct sales organization and distribution channel |
| Item usage | Special purpose of the item in the current document | Often blank for an ordinary stock item; relevant to specialized processing |
| Higher-level item category | Processing context supplied by a parent item | Relevant for subitems such as free goods or BOM components |

Common standard illustration:

```text
Sales document type:        OR
Item-category group:        NORM
Item usage:                 blank
Higher-level item category: blank
Result:                     TAN
```

The example is configuration-dependent. It illustrates the lookup; it is not a universal promise about every SAP system.

### Why the item-category group is insufficient by itself

The same material may need different processing in an inquiry, quotation, order, return, free-goods item, or subitem. The material classification supplies only one part of the business context. The document type, item usage, and parent-item context complete the lookup.

## What the item category controls

An item category can influence:

- whether pricing applies,
- whether and how the item is billed,
- whether item business data may differ from the header,
- whether the line represents a material or text item,
- which incompletion procedure is used,
- whether schedule lines are allowed, and
- processing of main items and subitems.

It is better to say that the inputs **select an item category**, and the selected category **controls the line's behavior**. Combining those two steps into one vague statement hides the most useful debugging distinction.

## Schedule-line-category determination

SAP uses a two-step search:

```text
1. Item category + material/plant MRP type
2. Item category + blank MRP type        ← fallback
────────────────────────────────────────
= Proposed schedule-line category
```

The MRP type is plant-specific material data, commonly inspected as `MARC-DISMM`. Always use the Plant from the affected item; a different Plant can mean a different MRP type and therefore a different determination result.

The assignment can also define schedule-line categories that a user may select as alternatives to the proposed one.

## What the schedule-line category controls

A schedule-line category can influence:

- delivery relevance,
- requirements transfer,
- availability checking,
- the movement type used by later goods movement,
- automatic purchase-requisition behavior, and
- schedule-line delivery blocks.

The item category must first permit schedule lines. If no schedule line exists, inspect that item-level permission before diagnosing schedule-line controls.

## Two nuances you must retain

### 1. Configured movement type does not mean inventory was posted

```text
Schedule-line configuration → defines later goods-movement behavior
Sales Order save            → records order demand and instructions
Post Goods Issue            → executes inventory movement
```

A movement type in schedule-line configuration tells downstream logistics how an applicable goods movement should behave. Saving the Sales Order does not execute PGI and is not proof of an inventory change. Look for the actual goods-movement/material-document evidence.

Also avoid the absolute statement that configuration “does not affect Sales Order data.” Configuration does influence processing, proposals, and control. The precise point is that **declaring a posting rule is not the same as executing a posting**.

### 2. Delivery relevance can be controlled at different levels

For normal items with schedule lines, the schedule-line category is central to determining whether each dated quantity is relevant for delivery. The delivery-relevance indicator in the item category specifically supports items without schedule lines, such as a delivery-relevant text item.

Therefore, “delivery relevance is always controlled only at schedule-line level” is too broad.

## Technical Consultant map

These are classic on-premise/private-edition landmarks. SAP S/4HANA Cloud Public Edition uses available configuration activities rather than assuming access to the same SAP GUI transactions. For clean-core development, prefer released APIs and CDS entities over raw-table dependencies.

| Purpose | Maintenance transaction | Configuration table landmark | Selected transaction result |
| --- | --- | --- | --- |
| Define sales document type | `VOV8` | `TVAK` | `VBAK-AUART` |
| Define item category behavior | `VOV7` | `TVAP` | `VBAP-PSTYV` |
| Assign/determine item category | `VOV4` | `T184` | Proposes `VBAP-PSTYV` |
| Define schedule-line category behavior | `VOV6` | `TVEP` | `VBEP-ETTYP` |
| Assign/determine schedule-line category | `VOV5` | `TVEPZ` | Proposes `VBEP-ETTYP` |

Master-data inputs:

```text
MVKE-MTPOS → material item-category group by sales area
MARC-DISMM → MRP type by material and Plant
```

Memory pattern:

```text
VOV8 defines the header process
VOV4 finds the item category; VOV7 defines it
VOV5 finds the schedule category; VOV6 defines it
```

## Full diagnostic algorithm

### Phase 1: capture the actual result

1. Open the Sales Order in `VA03`.
2. Identify the exact `VBELN`, `POSNR`, and, when applicable, `ETENR`.
3. State expected versus actual behavior.
4. Record the selected results:

```text
VBAK-AUART → sales document type
VBAP-PSTYV → item category
VBEP-ETTYP → schedule-line category
```

### Phase 2: diagnose the item category

1. Read the document type.
2. Read `MVKE-MTPOS` in the correct sales organization and distribution channel.
3. Check item usage.
4. If it is a subitem, check the higher-level item's category.
5. Reconstruct the exact `VOV4` assignment.
6. If the result is correct, inspect the selected category in `VOV7`.

### Phase 3: diagnose the schedule-line category

1. Read `VBEP-ETTYP` from the affected schedule line.
2. Read `VBAP-PSTYV` from its item.
3. Read `MARC-DISMM` for the correct material and Plant.
4. In `VOV5`, check the exact item-category/MRP-type combination.
5. If it has no match, check the item-category/blank-MRP fallback.
6. If the correct category was selected, inspect its requirements-transfer, ATP, delivery, and related controls in `VOV6`.

### Phase 4: widen the investigation

Only after standard inputs and configuration are explained should you widen the search to:

- copied reference-document data,
- missing or invalid master data,
- other dependent configuration,
- enhancements or custom code,
- integration payloads, or
- edition- and release-specific behavior.

Never directly edit transaction or configuration tables to repair the symptom. Use authorized configuration activities, document the rationale, test affected processes, and transport changes through the landscape.

## SD category determination versus RAP determination

| Concept | Purpose | Mechanism |
| --- | --- | --- |
| SD category determination | Select the configured category that fits the business inputs | Customizing lookup such as `VOV4` or `VOV5` |
| RAP determination | Automatically derive or update BO data after a declared trigger | Behavior definition plus behavior-pool implementation |
| RAP validation | Reject inconsistent BO state with a message | Validation declared in the behavior definition and implemented in the behavior pool |

Example:

```text
SD:  OR + NORM + blank + blank → select item category TAN
RAP: quantity or price changes → recalculate NetAmount
```

Both use the word “determination,” but they solve different problems.

## Where your answers needed reinforcement

### Business behavior versus selection logic

When asked what the item category controls, you initially described how business inputs combine to form a meaningful category. That describes **selection**. Keep the second half explicit: after selection, the item-category definition controls pricing, billing, text/material behavior, schedule-line permission, and other line behavior.

### Movement type versus actual posting

You correctly recognized configuration as processing logic, but the stronger technical answer names the execution boundary: the Sales Order records demand; PGI executes the inventory movement. A configured movement type is not posting evidence.

### Exact technical map versus general diagnostic flow

When asked specifically about sales document type, you first gave the whole `VBAK → VBAP → VBEP` investigation. That flow is useful, but interview questions often test one exact mapping:

```text
VOV8 → TVAK → VBAK-AUART
```

Answer the narrow question first, then add the broader flow.

### Assignment inputs must be precise

You correctly separated `VOV7` definition from `VOV4` assignment, but initially described `VOV4` as checking only order type and material group or assigning to a customer's order. The lookup is reusable configuration and uses four inputs—not the customer itself.

### Reverse tracing needs a repeatable algorithm

You needed hints for both wrong item-category and missing requirements-transfer scenarios. Your final answers were correct after using the formula backward. Retain this pattern:

```text
Stored result → determination inputs → assignment → definition → wider causes
```

## Polished answers to the ten conceptual questions

### 1. What does each control level answer?

The sales document type answers what kind of commercial transaction the whole document represents. The item category answers how one product or service line participates. The schedule-line category answers how one dated quantity participates in logistics and fulfillment.

### 2. Why can the same material behave differently in an inquiry and an order?

The sales document type is an input to item-category determination. Therefore, the same material and item-category group can receive different item categories in different document contexts, and those categories can enable different pricing, billing, delivery, or informational behavior.

### 3. Which inputs determine the item category?

SAP combines sales document type, material item-category group, item usage, and higher-level item category to propose an item category. Usage and higher-level category may be blank for a normal standalone item but matter for specialized items and subitems.

### 4. What can an item category control?

An item category can control whether pricing applies, whether and how the line is billed, whether its business data may differ from the header, whether it is a material or text item, its incompletion procedure, and whether schedule lines are permitted.

### 5. Why is the material item-category group insufficient?

It classifies the material but does not provide the complete transaction context. Document type, item usage, and higher-level item category allow the same material to participate differently in different sales processes or item hierarchies.

### 6. How is the schedule-line category determined?

SAP first searches for an assignment using item category plus the material's MRP type for the relevant Plant. If no match exists, it searches using the item category plus a blank MRP type.

### 7. What can a schedule-line category control?

It can control delivery relevance, requirements transfer, availability checking, the movement type for later inventory processing, automatic procurement behavior, and schedule-line delivery blocks.

### 8. Why does a configured movement type not change inventory when the order is saved?

The movement type defines how a later applicable goods movement should post. Saving the Sales Order records demand and processing instructions; inventory changes only when the logistics process executes the goods movement, normally at PGI.

### 9. Which level controls delivery relevance?

For items with schedule lines, the schedule-line category normally controls whether each dated quantity is delivery relevant. For items without schedule lines, the item-category delivery-relevance indicator can make the item relevant for delivery.

### 10. How does SD category determination differ from RAP determination?

SD category determination selects a configured category from business inputs. A RAP determination is behavior logic that automatically derives or updates BO data when a declared trigger occurs.

## Polished answers to the five technical checkpoints

### T1. Where is the sales document type defined and stored?

Use `VA03` to inspect the document. In classic configuration, `VOV8` maintains the sales document type definition, `TVAK` is its configuration-table landmark, and the selected transaction value is stored in `VBAK-AUART`.

### T2. How do `VOV7`/`TVAP` differ from `VOV4`/`T184`?

`VOV4`/`T184` hold the item-category determination assignment: the input combination selects an item category. `VOV7`/`TVAP` define what that selected item category does.

### T3. How do `VOV6`/`TVEP` differ from `VOV5`/`TVEPZ`?

`VOV5`/`TVEPZ` assign or determine a schedule-line category from item category and MRP type, including the blank-MRP fallback. `VOV6`/`TVEP` define how the selected schedule-line category behaves.

### T4. How do you diagnose the wrong item category?

First inspect the affected item and `VBAP-PSTYV` in `VA03`. Reconstruct the sales document type, material item-category group for the correct sales area, item usage, and higher-level item category. Verify that exact combination in `VOV4` before investigating enhancements.

### T5. How do you diagnose a schedule line that does not transfer requirements?

Inspect the item and schedule line in `VA03` and record `VBEP-ETTYP`. Reconstruct `VBAP-PSTYV` and `MARC-DISMM` for the correct Plant. Verify the exact and blank-MRP assignments in `VOV5`. If the expected category was selected, inspect its requirements-transfer and availability-check settings in `VOV6`.

## Active-recall keywords

### Three control levels

```text
Document type → whole transaction
Item category → one product/service line
Schedule category → one dated quantity
```

### Two determination formulas

```text
AUART + MTPOS + usage + higher-level category → PSTYV
PSTYV + DISMM → ETTYP
```

### Definition versus assignment

```text
VOV8: define document type
VOV4: determine item category
VOV7: define item category
VOV5: determine schedule category
VOV6: define schedule category
```

### Persistence and configuration landmarks

```text
VBAK-AUART ↔ VOV8 / TVAK
VBAP-PSTYV ↔ VOV4 / T184 → VOV7 / TVAP
VBEP-ETTYP ↔ VOV5 / TVEPZ → VOV6 / TVEP
MVKE-MTPOS = sales-area input
MARC-DISMM = material/Plant input
```

### Diagnostic mantra

```text
Grain
→ stored result
→ inputs
→ assignment
→ definition
→ wider causes
```

### Interview traps

```text
Selection ≠ behavior
Movement type ≠ executed posting
Order saved ≠ PGI posted
Schedule-line delivery relevance ≠ every item case
SD determination ≠ RAP determination
Configuration table ≠ direct-edit target
Classic table ≠ clean-core integration contract
```

## Spaced active-recall drill

Without opening this note, answer these prompts tomorrow, in three days, and in one week:

1. Draw the three control levels and state one behavior controlled at each.
2. Write the four-input item-category formula.
3. Write both schedule-line determination attempts in order.
4. Explain assignment versus definition using the `VOV4`/`VOV7` pair.
5. Map `VOV5`, `VOV6`, `TVEPZ`, `TVEP`, and `VBEP-ETTYP`.
6. Explain why movement type `601` in configuration does not prove PGI.
7. Diagnose one wrong item-category case using the reverse-trace algorithm.
8. Diagnose one correct-category-but-wrong-behavior case.

## Primary sources

- [SAP Help — How Sales Documents Are Controlled](https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/7b24a64d9d0941bda1afa753263d9e39/c264b65334e6b54ce10000000a174cb4.html)
- [SAP Learning — Configuring a Sales Document Type](https://learning.sap.com/courses/fundamental-customizing-in-sap-s-4hana-sales/configuring-a-sales-document-type)
- [SAP Learning — Configuring an Item Category](https://learning.sap.com/courses/fundamental-customizing-in-sap-s-4hana-sales/configuring-an-item-category-1)
- [SAP Learning — Configuring Schedule Line Categories](https://learning.sap.com/courses/fundamental-customizing-in-sap-s-4hana-sales/configuring-schedule-line-categories)
- [SAP Learning — Assigning Schedule Line Categories](https://learning.sap.com/courses/fundamental-customizing-in-sap-s-4hana-sales/assigning-schedule-line-categories)

## Readiness statement

You can now explain the three-level Sales document control model and reconstruct both category lookups. Your next growth area is speed and independence: practice starting from the stored result and naming every input without hints. This prepares you to learn pricing determination through the condition technique without confusing a selected procedure or condition type with the behavior configured behind it.
