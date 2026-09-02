# Lesson 11 Recap: SAP SD Order-to-Cash Document Flow

## Lesson objective

Trace a sale from customer interest to financial settlement and explain, for every stage:

1. the business intent,
2. the document or posting,
3. the responsible role,
4. the operational or financial effect, and
5. what has **not** happened yet.

The goal is not to memorize transaction codes. The goal is to understand why each document exists and what new business fact it records.

## The whole lesson in one sentence

Order-to-cash is a chain of related but independently meaningful business documents; each document records a different commitment, execution event, or financial event.

## Core process

```text
Optional presales                 Core order-to-cash execution

Inquiry → Quotation → Sales Order → Outbound Delivery → PGI → Billing → Payment
```

Another useful representation is:

```text
Interest → Offer → Commitment → Fulfillment preparation
         → Physical movement → Customer debt → Settlement
```

Inquiry and quotation are optional presales documents. The central order-to-cash process normally concerns the accepted customer order, fulfillment, billing, and payment.

## Document, owner, and business effect

| Stage | Main owner | What becomes true | What has not happened yet |
|---|---|---|---|
| Inquiry | Sales | The customer's request for information is recorded. | No commercial offer or sale is agreed. |
| Quotation | Sales | A formal, usually time-limited commercial offer exists. | The customer has not necessarily placed an order. |
| Sales Order | Sales | Products, quantities, partners, prices, dates, and fulfillment intent are recorded. | Goods have not physically left inventory. |
| Outbound Delivery | Shipping | The warehouse has an execution document for delivery, picking, and optional packing. | Goods issue has not necessarily been posted. |
| Post Goods Issue | Shipping / Inventory | The system records the physical departure of the goods; inventory quantity and value are affected. | The customer receivable is not created by PGI. |
| Billing Document | Billing | The customer is invoiced; accounting normally records receivable and revenue effects. | The company has not necessarily received the money. |
| Incoming Payment | Finance | The received money is posted and matched against an open receivable. | Other deliveries, invoices, or open items may still remain. |

## The most important insights

### 1. A business process is not one document

A Sales Order is one document inside order-to-cash. It records the commercial and fulfillment intent, but it does not itself prove that the warehouse shipped goods, that the customer was invoiced, or that payment was received.

### 2. Document creation and decisive posting are different

Creating an Outbound Delivery gives Shipping a document from which to execute picking and packing. Posting Goods Issue is the event that records the physical inventory movement.

Therefore:

```text
Delivery created ≠ goods physically issued
Billing created  ≠ money received
```

### 3. The process crosses ownership boundaries

Different teams own different truths:

- Sales owns the offer and customer order.
- Shipping owns delivery execution.
- Inventory is affected at PGI.
- Billing creates the invoice.
- Finance records the receivable and incoming payment.

This separation lets each document have its own responsibility, controls, status, and audit history.

### 4. Quantities reveal process progress

One order quantity can split across dates and documents. For an order of 10 units with only 6 fulfilled:

```text
Ordered:   10
Delivered:  6
Billed:     6
Open:       4
```

The four units remain open for later fulfillment, assuming they were not rejected or cancelled.

### 5. Document flow is business lineage

Document flow answers: “Which earlier document caused this document, and what happened afterward?” It can link quotations, orders, deliveries, goods movements, invoices, accounting documents, and other successors.

A single status such as `Partially Completed` cannot explain the process as precisely as the linked documents and quantities.

### 6. SD document flow is not a RAP composition

A RAP composition models ownership inside one business-object aggregate. A child such as an Order Item depends on its root for lifecycle and transactional consistency.

An SD document flow links separate business documents across process stages. A Billing Document is not merely a lifecycle-dependent child of the Sales Order. It has its own purpose, data, status, responsible role, and subsequent processing.

## Sales Order structure

### Header

The header contains information that applies to the whole Sales Order.

Examples:

- sold-to party,
- sales area,
- document currency,
- order date, and
- overall commercial or delivery information.

Question answered: **Who is this transaction with, and under which commercial context?**

### Item

An item represents one product or service line.

Examples:

- material,
- ordered quantity,
- item category,
- delivering plant, and
- pricing conditions.

Question answered: **What is being sold, and how should this particular line be processed?**

### Schedule line

A schedule line represents a quantity and date commitment within an item.

Examples:

- confirmed quantity,
- requested delivery date, and
- confirmed delivery date.

Question answered: **How much can be delivered, and when?**

One item can have multiple schedule lines when its quantity is confirmed for different dates.

## Corrections to remember

These were the most useful refinements from the retrieval session.

| Initial idea | More precise understanding |
|---|---|
| Order-to-cash always starts with customer interest. | Inquiry and quotation are optional presales stages; core order-to-cash normally centers on the accepted order through settlement. |
| A Sales Order only confirms order information. | It records the commercial and fulfillment intent and becomes a basis for subsequent processing. |
| Outbound Delivery is only a planning step. | It is a shipping execution document controlling delivery, picking, and optional packing. |
| PGI evaluates inventory after shipping. | PGI **posts** the physical goods movement and affects inventory quantity and value. |
| A Billing Document only has meaning for its Sales Order. | Billing is a separate business document and normally transfers receivable and revenue effects to FI. |
| Payment sends order information back to Finance. | Finance posts the received money and clears the corresponding open receivable. |
| Payment always means the complete order is finished. | Other partial deliveries, invoices, disputes, or open items may still exist. |
| Two unavailable units are stored for later. | They remain open or backordered until later confirmation and fulfillment. |
| RAP composition is only a technical entity relationship. | It expresses business ownership and lifecycle dependency inside one transactional aggregate. |

## Polished answers to all ten retrieval questions

### 1. What is the difference between order-to-cash and a single Sales Order?

Order-to-cash is the end-to-end business process covering the customer order, fulfillment, goods issue, billing, and payment. A Sales Order is one document within that process. It records the agreed products, quantities, partners, prices, and delivery requirements and provides the basis for subsequent fulfillment.

### 2. Which documents may optionally appear before the Sales Order?

An **Inquiry** records a customer's request for information about products, availability, prices, or delivery conditions. It does not represent an agreed sale.

A **Quotation** is a formal commercial offer, usually containing products, quantities, prices, terms, and a validity period. It can later be referenced when creating a Sales Order.

### 3. What does a Sales Order record, and what physical event has not happened?

A Sales Order records products or services, quantities, business partners, prices, requested delivery dates, and shipping requirements. It represents commercial and fulfillment intent, but the goods have not physically left the company and PGI has not reduced inventory.

### 4. Why are Outbound Delivery and PGI separate?

The Outbound Delivery is the shipping execution document. It contains delivery quantities, shipping data, and information for picking and packing. Creating it prepares and controls fulfillment.

PGI records that the goods physically left. It posts the inventory movement and affects inventory quantity and value. The separation allows warehouse work to be prepared and checked before the irreversible business event is recorded.

### 5. Which event changes inventory quantity and value?

Posting Goods Issue changes inventory quantity and value because it records the physical departure of the goods. A Sales Order records demand, while an Outbound Delivery controls shipping execution. Neither document alone posts the physical inventory movement.

### 6. How do billing and incoming payment differ?

A Billing Document invoices the customer and transfers the relevant values to Financial Accounting, normally creating a customer receivable and revenue posting. The customer now owes the company money.

An incoming payment records that money was received. Finance matches the payment against the open receivable and clears it. Payment does not necessarily finish every part of the order-to-cash flow because other deliveries, invoices, or open items may remain.

### 7. What belongs at header, item, and schedule-line level?

The **header** contains data applying to the entire Sales Order, such as the sold-to party, sales area, document currency, and overall dates or terms.

An **item** represents one product or service line and contains data such as the material, ordered quantity, item category, delivering plant, and pricing conditions.

A **schedule line** contains a quantity/date commitment within an item, such as confirmed quantity and confirmed delivery date.

### 8. Why can one Sales Order item produce multiple downstream documents?

The complete ordered quantity may not be available on the same date. If a customer orders 10 units but only 8 are available now, one schedule line may confirm 8 for the first date and another may confirm 2 for a later date.

This can produce separate deliveries for 8 and 2 units. Depending on the billing process and configuration, the deliveries may also lead to separate billing documents. The Sales Order item retains the original quantity and tracks fulfilled and open quantities.

### 9. How is SD document flow different from RAP composition?

An SD document flow links separate business documents created at different process stages. Each document has its own purpose, lifecycle, status, and responsible business role.

A RAP composition defines ownership inside one business-object aggregate. A composition child is lifecycle-dependent on its root and participates in the root's transactional consistency boundary.

Therefore, the entire SD process should not be modeled as one large RAP composition. A Billing Document is a successor in the document flow, not simply a child entity owned by the Sales Order.

### 10. Six of ten ordered units were delivered, PGI-posted, and billed. What is the current quantity state?

The ordered quantity remains 10. The delivered and PGI-posted quantity is 6, and the billed quantity is 6. Four units remain open for later fulfillment, assuming there was no rejection or cancellation.

```text
Ordered:   10
Delivered:  6
Billed:     6
Open:       4
```

## Active-recall keywords

Use only the left column at first. Hide the explanation and reconstruct it aloud.

| Cue | What to retrieve |
|---|---|
| O2C scope | End-to-end process, not one order document |
| Inquiry | Interest or information request; no agreed sale |
| Quotation | Formal offer; terms and validity |
| Sales Order | Commercial intent and fulfillment basis; no physical issue |
| Outbound Delivery | Shipping execution; picking and packing |
| PGI | Physical departure; inventory quantity and value |
| Billing | Customer invoice; receivable and revenue |
| Payment | Cash received; open receivable cleared |
| Header | Whole-document commercial context |
| Item | One product/service processing line |
| Schedule line | Confirmed quantity plus delivery date |
| Partial fulfillment | One item, split dates and downstream documents |
| Open quantity | Ordered minus fulfilled, subject to rejection/cancellation |
| Document flow | Cross-document lineage and process history |
| RAP composition | Root ownership and dependent child lifecycle |
| Delivery ≠ PGI | Execution document versus posted inventory event |
| Billing ≠ payment | Customer debt versus financial settlement |

## Ten-second cues

If you have very little time, recall these contrasts:

```text
Process       ≠ one document
Order         ≠ shipment
Delivery      ≠ goods issue
Billing       ≠ payment
Status        ≠ document lineage
Document flow ≠ composition
```

## Ninety-second interview answer

> In a standard stock-sales order-to-cash process, Sales records the customer order, including partners, items, quantities, prices, and requested dates. Shipping then creates an Outbound Delivery to control picking and packing. Creating the delivery does not reduce stock; Posting Goods Issue records that the goods physically left and affects inventory quantity and value. Billing then invoices the customer and normally creates the receivable and revenue effects in Finance. Incoming payment is a separate Finance event that clears the receivable. These are separate but linked documents because each stage has a different business owner, lifecycle, control point, and operational or financial effect. Document flow preserves that lineage, including partial deliveries and invoices.

## Spaced active-recall plan

Review the lesson without reading this file first:

- **Tomorrow:** draw the seven-stage flow and explain each “not yet” boundary.
- **After 3 days:** answer all ten questions in random order.
- **After 7 days:** explain the partial-delivery scenario and compare document flow with RAP composition.
- **After 14 days:** deliver the ninety-second answer, then check this note for missing effects.

## Sources

- [Lesson 11 — Trace the SD Order-to-Cash Flow](../lessons/0011-trace-sd-order-to-cash.html)
- [SAP Learning — Navigating the Order-to-Cash Process Steps](https://learning.sap.com/courses/discovering-the-basics-of-sap-s-4hana-sales/executing-sales-order-management_cad9dfbe-bafc-4ed9-ac2e-6fd8422430e4)
- [SAP Learning — Sales Order and Receivables Integration](https://learning.sap.com/courses/end-to-end-processes-in-receivables-management/explaining-the-integration-between-sales-order-and-receivables-management_b97d8b06-c206-43cf-a6b2-b9bf1f3ccf37)
- [Complete SAP SD Source Map](../reference/sap-sd-source-map.html)
