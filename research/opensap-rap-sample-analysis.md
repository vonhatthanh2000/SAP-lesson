# SAP openSAP RAP sample: architecture and lesson map

Verified: 2026-08-18  
Repository: [`SAP-samples/abap-platform-rap-opensap`](https://github.com/SAP-samples/abap-platform-rap-opensap)  
Default branch: `main`  
Pinned revision: [`71f57c5a1fc1cf902bd38afb1259f79610732ce6`](https://github.com/SAP-samples/abap-platform-rap-opensap/tree/71f57c5a1fc1cf902bd38afb1259f79610732ce6) (2026-06-03)

## Executive conclusion

This repository is useful as a **single, progressive architecture case study**. It starts with a read-only Travel application, turns it into a managed transactional RAP business object, and then rebuilds a similar business object as unmanaged around legacy function modules. That makes it especially good for seeing how artifacts connect.

It is not a current syntax authority. SAP explicitly says the associated 2020 openSAP course is no longer available and the exercises are not up to date ([README, lines 4–18](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/README.md#L4-L18)). Use current SAP RAP documentation and newer RAP100/RAP110 samples to confirm syntax and production guidance.

For the current curriculum, embed only two small slices:

1. Lesson 1: trace `acceptTravel` from Fiori metadata to its behavior implementation and then to the transactional save boundary.
2. Lesson 2: analyze why `Travel` is the root, `Booking` is a composition child, and Agency/Customer/Flight objects are associations.

Defer draft, detailed business logic, security, EML, save sequence, and unmanaged implementation until their dedicated lessons. Loading all of those into Lessons 1–2 would hide the two mental models those lessons are intended to establish.

## The actual sample domain

The managed example models a travel-booking aggregate:

```text
Travel (root)
├── owns 0..* Booking children
├── references Agency
├── references Customer
└── references Currency

Booking (child)
├── belongs to one Travel
├── references Customer
├── references Carrier / Connection / Flight
└── references Currency
```

`Travel` carries dates, booking fee, total price, status, description, and audit fields. `Booking` carries the selected flight, passenger/customer, flight price, and currency. Their actual storage is shown in the Travel and Booking table definitions ([Travel table, lines 6–25](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U2_TABL_ZRAP_ATRAV.txt#L6-L25), [Booking table, lines 6–21](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U2_TABL_ZRAP_ABOOK.txt#L6-L21)).

The sample is a `/DMO/FLIGHT` teaching domain, not SAP SD. The useful transfer is structural:

| Travel sample | Sales-order portfolio |
|---|---|
| Travel root | SalesOrderRequest root |
| Booking composition child | SalesOrderItem composition child |
| Agency / Customer reference | Sold-to party association |
| Carrier / Connection / Flight reference | Product association and other external master-data references |
| `acceptTravel` action | `Release` action |
| `TravelStatus` | Order workflow status |
| `TotalPrice` | Order total |

## Lesson 1 analysis: trace one request end to end

### The `Accept Travel` vertical slice

Use this path in the lesson:

```text
Fiori elements button
  -> OData UI service binding
  -> service definition
  -> projection CDS + projected behavior
  -> base behavior definition
  -> behavior handler action
  -> RAP transactional buffer
  -> save sequence
  -> active persistence table
```

1. The metadata extension asks Fiori elements to render `acceptTravel` and `rejectTravel` as actions ([UI metadata, lines 60–68](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U5_DDLX_ZC_RAP_TRAVEL.txt#L60-L68)). This is presentation metadata; it does not declare or implement the business operation.
2. The repository does not contain a serialized service-binding source file. The exercise tells the learner to create `ZUI_RAP_TRAVEL_O2_####`, choose **OData V2 - UI**, activate it, and publish it ([Week 2 Unit 6, lines 82–107](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/unit6.md#L82-L107)).
3. The service definition exposes the projected Travel and Booking entities, plus value-help entities ([service definition, lines 1–13](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U6_SRVD_ZUI_RAP_TRAVEL.txt#L1-L13)).
4. The projection CDS chooses consumer-facing fields and redirects `_Booking` to the projected child ([Travel projection, lines 6–40](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U4_DDLS_ZC_RAP_TRAVEL.txt#L6-L40)). The behavior projection explicitly exposes the two actions with `use action` ([projected BDEF, lines 1–22](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U5_BDEF_ZC_RAP_TRAVEL.txt#L1-L22)).
5. The base behavior definition declares `acceptTravel` and `rejectTravel`, their instance feature control, determinations, validations, persistence mappings, locking, ETags, and authorization responsibility ([base BDEF, lines 1–51](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U6_BDEF_ZI_RAP_TRAVEL.txt#L1-L51)).
6. The handler implements `acceptTravel` by updating `TravelStatus` in local mode and reading the changed entity back as the action result ([handler, lines 247–268](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_CLAS_ZBP_I_RAP_TRAVEL.txt#L247-L268)). `IN LOCAL MODE` operates through the BO's transaction, rather than bypassing the BO contract with a direct table update.
7. The EML exercise makes the interaction/save boundary visible: `MODIFY ENTITIES` performs an update, while `COMMIT ENTITIES` triggers completion of the transaction ([EML update and commit, lines 63–80](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U4_CLAS_ZCL_RAP_EML.txt#L63-L80)). In the managed BO, RAP owns standard persistence to the table declared and mapped in the base BDEF.

### The critical teaching correction

Do **not** present `acceptTravel` as a complete implementation of the portfolio's `Release` invariant.

The sample's feature-control method disables `acceptTravel` only when the Travel is already accepted ([handler, lines 295–317](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_CLAS_ZBP_I_RAP_TRAVEL.txt#L295-L317)), but the action implementation itself unconditionally writes the accepted status. A non-UI consumer can invoke actions through the BO contract. Therefore the Sales Order `Release` implementation must check the current state and report failure in the backend; feature control remains a UI/consumer affordance, not the invariant.

### Exact Lesson 1 addition

Add one compact section titled **“Code trace: Accept Travel”** containing:

- the seven-step path above;
- three highlighted responsibility checks: base BDEF **declares**, behavior projection **exposes**, handler **implements**;
- the warning that the sample action does not enforce a state transition in its backend implementation;
- a 10-minute exercise: mark each of the seven artifacts with `presentation`, `consumer contract`, `BO contract`, `implementation`, or `persistence boundary`, then redraw the same trace for `Release`.

Do not copy whole source listings into the HTML. Show only small excerpts around `use action`, the base action declaration, and `METHOD acceptTravel`, linked to the pinned files.

## Lesson 2 analysis: find the aggregate boundary in code

### Evidence for root, child, and references

- `ZI_RAP_Travel_####` is declared as a **root view entity** and owns `composition [0..*] of ZI_RAP_Booking_####` ([Travel interface CDS, lines 1–10](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U3_DDLS_ZI_RAP_TRAVEL.txt#L1-L10)). Agency, Customer, and Currency are ordinary associations because those referenced objects exist independently of a Travel.
- `ZI_RAP_Booking_####` declares `association to parent` back to Travel, while Customer, Carrier, Connection, Flight, and Currency remain ordinary associations ([Booking interface CDS, lines 1–16](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U3_DDLS_ZI_RAP_BOOKING.txt#L1-L16)).
- The projection preserves the topology by redirecting `_Booking` to the projected composition child ([Travel projection, lines 35–39](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U4_DDLS_ZC_RAP_TRAVEL.txt#L35-L39)) and `_Travel` to the projected parent ([Booking projection, lines 36–41](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U4_DDLS_ZC_RAP_BOOKING.txt#L36-L41)).
- The behavior contract makes ownership operational: the root permits create-by-association for `_Booking`; Booking's lock and authorization are dependent on Travel; Booking's `TravelUUID` is read-only ([base BDEF, lines 3–18 and 53–70](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U6_BDEF_ZI_RAP_TRAVEL.txt#L3-L18)).
- Aggregate-wide calculation crosses the composition boundary: the handler reads a Travel's Bookings, sums their prices (with currency conversion), and updates the root total ([total calculation, lines 455–516](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_CLAS_ZBP_I_RAP_TRAVEL.txt#L455-L516)). This is behavioral evidence that Travel is the consistency boundary, not merely a table with a foreign key.

### Exact Lesson 2 addition

Add one compact section titled **“Code reading: Travel owns Booking”** containing:

1. the five evidence bullets above, shortened to one sentence each;
2. a side-by-side translation: `Travel -> SalesOrderRequest`, `Booking -> Item`, `Customer/Flight -> Customer/Product`;
3. three closed-book questions:
   - Which two CDS declarations establish the parent-child topology?
   - Why is Customer an association even though Travel cannot be valid without one?
   - Which BDEF statements turn lifecycle ownership into allowed operations and dependent policies?

The implementation task remains the learner's own Sales Order diagram and invariants. Do not ask them to copy the Travel CDS; the point is to transfer the lifecycle reasoning.

## Exact code-object index

| Notion | Best pinned sample | What it demonstrates |
|---|---|---|
| Persistence | [Travel table](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U2_TABL_ZRAP_ATRAV.txt#L6-L25), [Booking table](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U2_TABL_ZRAP_ABOOK.txt#L6-L21) | Managed example's active storage and technical keys |
| Root CDS and references | [Travel interface CDS](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U3_DDLS_ZI_RAP_TRAVEL.txt#L1-L40) | Root, composition, associations, semantic fields |
| Child and parent | [Booking interface CDS](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U3_DDLS_ZI_RAP_BOOKING.txt#L1-L43) | `association to parent` and independent references |
| Managed BO | [Managed base BDEF](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U6_BDEF_ZI_RAP_TRAVEL.txt#L1-L88) | RAP-owned standard CRUD/persistence plus custom behavior hooks |
| Base behavior definition | [Travel section](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U6_BDEF_ZI_RAP_TRAVEL.txt#L3-L51) | Contract, lock, authorization, ETag, actions, determinations, validations, mapping |
| Behavior implementation | [Travel handler](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_CLAS_ZBP_I_RAP_TRAVEL.txt#L1-L54) | Handler method declarations corresponding to BDEF hooks |
| Projection CDS | [Travel projection](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U4_DDLS_ZC_RAP_TRAVEL.txt#L1-L40), [Booking projection](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U4_DDLS_ZC_RAP_BOOKING.txt#L1-L43) | Consumer-facing fields, value helps, redirected topology |
| Behavior projection | [Projected BDEF](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U5_BDEF_ZC_RAP_TRAVEL.txt#L1-L23) | Consumer-visible CRUD, association operations, and actions |
| Service definition | [UI service definition](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U6_SRVD_ZUI_RAP_TRAVEL.txt#L1-L13) | Projected entities included in the business service |
| Service binding | [Interactive creation instructions](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/unit6.md#L82-L107) | Binding the service definition to the then-supported OData V2 UI protocol |
| UI metadata | [Travel metadata extension](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U5_DDLX_ZC_RAP_TRAVEL.txt#L1-L74) | Header/facets/fields and action buttons for Fiori elements |
| Draft | [Draft base BDEF](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_BDEF_ZI_RAP_TRAVEL.txt#L1-L38), [draft projection](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_BDEF_ZC_RAP_TRAVEL.txt#L1-L24) | Draft tables, total ETag, draft-enabled associations, Prepare validations, `use draft` |
| Validations | [BDEF triggers](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_BDEF_ZI_RAP_TRAVEL.txt#L26-L38), [`validateDates`](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_CLAS_ZBP_I_RAP_TRAVEL.txt#L208-L244) | On-save trigger and `FAILED`/`REPORTED` state messages |
| Determinations | [BDEF triggers](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_BDEF_ZI_RAP_TRAVEL.txt#L26-L28), [`setInitialStatus`](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_CLAS_ZBP_I_RAP_TRAVEL.txt#L96-L118) | On-modify defaulting; on-save readable ID calculation is another example |
| Actions | [Declaration](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_BDEF_ZI_RAP_TRAVEL.txt#L22-L24), [`acceptTravel`](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_CLAS_ZBP_I_RAP_TRAVEL.txt#L247-L268) | Declared operation versus implementation |
| Feature control | [`get_features`](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_CLAS_ZBP_I_RAP_TRAVEL.txt#L295-L317) | Instance-state-dependent enablement of actions |
| RAP authorization | [Authorization contract](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U6_BDEF_ZI_RAP_TRAVEL.txt#L3-L8), [`get_authorizations`](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_CLAS_ZBP_I_RAP_TRAVEL.txt#L320-L411) | Instance authorization requested and returned per operation |
| CDS access control | [DCL role](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U7_DCLS_ZI_RAP_TRAVEL.txt#L1-L14) | Read restriction using a PFCG aspect and currency predicate |
| EML | [Read and read-by-association](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U4_CLAS_ZCL_RAP_EML.txt#L17-L61), [modify/create and commit](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U4_CLAS_ZCL_RAP_EML.txt#L63-L105) | BO consumption without the UI; `MAPPED`, `FAILED`, `REPORTED`, commit boundary |
| Save sequence | [Managed EML commit](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U4_CLAS_ZCL_RAP_EML.txt#L74-L78), [unmanaged saver](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week4/sources/W4U3_CLAS_ZBP_I_RAP_TRAVEL_U_%23%23%23%23.txt#L431-L452) | Consumer commit versus provider saver calling legacy persistence |
| Unmanaged BO | [Unmanaged BDEF](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week4/sources/W4U3_BDEF_ZI_RAP_TRAVEL_U_%23%23%23%23.txt#L1-L60), [handler create](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week4/sources/W4U3_CLAS_ZBP_I_RAP_TRAVEL_U_%23%23%23%23.txt#L29-L76) | RAP adapter over existing tables, function modules, buffer, locks, and save logic |
| Behavior test | [Unmanaged EML test](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week4/sources/W4U3_CLAS_TEST_ZBP_I_RAP_TRAVEL_U_%23%23%23%23.txt#L23-L90) | Create, commit, CDS test environment, and rollback |

## Managed versus unmanaged in this repository

The managed path owns custom `ZRAP_*` tables and declares `managed;`, persistent tables, managed UUID numbering, and field mappings. RAP supplies the standard transactional infrastructure while handler methods supply business-specific logic ([managed BDEF, lines 1–32](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U6_BDEF_ZI_RAP_TRAVEL.txt#L1-L32)).

The unmanaged path reads existing `/DMO/TRAVEL` and `/DMO/BOOKING` tables and deliberately keeps `/DMO/FLIGHT_TRAVEL_*` function modules as the transaction owner. SAP's scenario description says those functions create/update/delete in a shared legacy buffer and a separate function saves that buffer to the database ([Week 4 README, lines 6–19](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week4/README.md#L6-L19)). The handler maps RAP entities to the legacy API and fills `MAPPED`, `FAILED`, and `REPORTED`; the saver calls `/DMO/FLIGHT_TRAVEL_SAVE` ([unmanaged create, lines 29–70](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week4/sources/W4U3_CLAS_ZBP_I_RAP_TRAVEL_U_%23%23%23%23.txt#L29-L70), [saver, lines 431–452](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week4/sources/W4U3_CLAS_ZBP_I_RAP_TRAVEL_U_%23%23%23%23.txt#L431-L452)).

This is the repository's cleanest demonstration of the decision rule: managed when RAP owns the new persistence transaction; unmanaged when existing transactional logic must remain authoritative.

## Future-lesson routing

| Portfolio lesson | openSAP unit/code to analyze later | Intended question |
|---|---|---|
| 3. Projection and service | Week 2 Units 4–6: projection CDS, metadata extensions, service definition, interactive binding | How is one core model shaped and exposed for a specific consumer? |
| 4. Managed behavior and buffer | Week 3 Units 2 and 4: managed BDEF and EML | What does RAP own, and what is visible before/after `COMMIT ENTITIES`? |
| 5. Draft, locking, ETags, numbering | Week 3 Unit 7 draft BDEF; `CalculateTravelID` warning | Which state is draft/active, how is concurrency controlled, and why is `MAX + 1` not robust numbering? |
| 6. Determinations, validations, actions | Week 3 Units 5–6 | Which rule is automatic, which blocks save, and which is explicitly invoked? |
| 7. Feature control, messages, side effects | `get_features`, validation `FAILED`/`REPORTED`; supplement side effects from newer samples | How does consumer affordance differ from backend correctness? |
| 8. EML | Week 3 Unit 4 and handler-local EML | How do UI-independent consumers and BO-internal logic use the same contract? |
| 9. Authorization and access control | Week 2 Unit 7 DCL; Week 3 Unit 6 authorization | Which rows may be read, and which operations may be performed? |
| 10. Testing and debugging | Week 4 supplied behavior-pool test class; supplement with RAP100/110 tests | How is behavior verified below the UI? |
| 11. Managed/unmanaged and clean core | Week 4 Units 2–5 | When should RAP wrap an existing transaction engine rather than persist directly? |
| 12. Portfolio/API defense | Week 5 Units 2–7: service consumption, custom entity, UI service and Web API | How should external data and separate consumer contracts be integrated? |

## Limitations and safety notes

1. **Outdated course:** SAP labels the exercises not up to date. Confirm all syntax and recommended patterns with current official RAP docs and RAP100/RAP110.
2. **OData generation:** the original UI exercise selects OData V2 and explicitly says V4 was only planned at the time ([Week 2 Unit 6, lines 82–93](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/unit6.md#L82-L93)). The portfolio should use its target environment's current supported binding, normally OData V4 where appropriate.
3. **No binding source artifact:** service bindings are created interactively in the exercise, so cite the instructions rather than implying there is a `.srvb` source file in the repository.
4. **Action invariant gap:** `acceptTravel` updates status without checking the current status. Treat it as action mechanics, not as a production workflow invariant.
5. **Authorization is deliberately bypassed:** the helpers perform `AUTHORITY-CHECK` and then force the result to `abap_true` for testing ([handler, lines 413–453](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_CLAS_ZBP_I_RAP_TRAVEL.txt#L413-L453)). Never copy that override into the portfolio.
6. **DCL can be neutralized in the exercise:** an adjusted role comments out the PFCG/currency condition and uses `where true` ([adjusted DCL, lines 1–15](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week2/sources/W2U7_DCLS_ZI_RAP_TRAVEL_Adjusted.txt#L1-L15)). It is a setup convenience, not authorization.
7. **Unsafe readable numbering:** the handler itself warns that selecting `MAX( travel_id )` does not guarantee unique or gap-free IDs ([handler, lines 59–80](https://github.com/SAP-samples/abap-platform-rap-opensap/blob/71f57c5a1fc1cf902bd38afb1259f79610732ce6/week3/sources/W3U7_CLAS_ZBP_I_RAP_TRAVEL.txt#L59-L80)). Keep UUID as the technical key and study current numbering options separately.
8. **Progressive snapshots:** several files represent different weekly stages of the same object. Use Week 3 Unit 7 as the final managed behavior snapshot, not every BDEF file as a separate design.
9. **Placeholder names:** `####` must be replaced with a learner-specific suffix. Links containing the placeholder encode `#` as `%23` where required.
10. **Coverage gaps:** the repository is strong for artifact flow, EML, draft, and classic managed/unmanaged comparison. Use newer samples/current docs for current side-effect syntax, production authorization, late numbering, current save variants, and modern OData V4 guidance.
11. **No side-effects example:** a repository-wide source search finds no RAP `side effects` declaration. Do not attribute that concept to this sample; introduce it from a newer first-party sample in the dedicated lesson.
