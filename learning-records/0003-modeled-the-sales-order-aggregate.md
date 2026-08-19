# Modeled the Sales Order aggregate boundary

The learner correctly modeled `SalesOrderRequest` as the sole aggregate root, `Item` as a composition child, and Customer/Product as independent associations. They also distinguished entered from derived amounts and stated testable invariants that require both header and item state.

## Evidence

From memory, the learner corrected the root/child topology, explained dependent lock and authorization, labeled `UnitPrice` as entered and `NetAmount`/`TotalAmount` as derived, and defined submission, total, and item-validity invariants.

## Implications

Future lessons may assume lifecycle-based aggregate modeling and move into CDS view entities, semantic annotations, and consumer projections. Continue testing whether the learner can distinguish a reusable BO contract from a service-specific projection.
