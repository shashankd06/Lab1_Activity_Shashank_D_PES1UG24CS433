# Exception Flow Specification Document
**Course:** PES University - Dept. of CSE  
**Lab 1:** Requirements Engineering & UML Use-Case Modelling  
**Name:** Shashank D  
**SRN:** PES1UG24CS433  
**Problem Statement #05:** Academic Elective Bidding & Allocation System  

---

## 1. Document Overview & Scope

In Software Engineering and UML Use-Case Modelling, **Exception Flows** define system behavior when non-standard, erroneous, or unexpected events occur during system interaction. Unlike **Alternate Flows**—which represent valid optional or secondary paths to success—**Exception Flows** address condition failures, invalid inputs, constraint violations, deadline lockouts, security breaches, and hardware/software execution errors.

This document provides a complete specification of all exception flows across the **Academic Elective Bidding & Allocation System** lifecycle, establishing precise error triggers, system validation responses, rollback mechanisms, user remediation paths, and security audit log invariants.

---

## 2. Exception Flow Summary Matrix

| Exception ID | Target Use Case | Exception Title | Error Category | Severity | Detection Trigger |
| :--- | :--- | :--- | :--- | :---: | :--- |
| **EF-01** | UC-01 | Credit Allocation Mismatch ($\neq 100$) | Data Validation | High | Form submission with total credit sum $\neq 100$. |
| **EF-02** | UC-01 / UC-02 | Unmet Course Prerequisite | Domain Rule | High | Automated query detects missing prerequisite completion record. |
| **EF-03** | UC-01 / UC-03 | Unresolvable Slot Collision | Hard Constraint | High | All submitted elective preferences overlap with core lecture/lab slots. |
| **EF-04** | UC-01 | Bidding Window Expiry Lockout | Temporal / State | Critical | Transaction commit timestamp exceeds strict portal bidding deadline. |
| **EF-05** | UC-05 | Solver Algorithm Timeout / Divergence | System Performance | Critical | Multi-criteria optimization engine execution exceeds 30-second ceiling. |
| **EF-06** | UC-01 / UC-05 | Database Concurrency Deadlock | Infrastructure | High | Simultaneous peak bidding commits trigger DB lock conflict or socket reset. |
| **EF-07** | UC-07 | Unauthorized Override Breach | Security / RBAC | Critical | Non-registrar account attempts manual allocation or audit tampering. |

---

## 3. Detailed Exception Specifications

### Exception EF-01: Credit Allocation Mismatch ($\neq 100$ Bidding Credits)
* **Associated Use Case:** UC-01 (Submit & Manage Elective Bids)
* **Primary Actor:** Student
* **Trigger Condition:** Student clicks "Submit Bids" when the sum of allocated bidding points across all selected electives is less than or greater than 100.

#### Step-by-Step Flow:
1. **System** interceptor calculates $\text{Sum}(\text{Bidding Points})$.
2. **System** detects $\text{Sum} \neq 100$ (e.g., Student allocated 90 or 110 total points).
3. **System** halts the commit transaction and blocks database write operations.
4. **System** highlights input fields with visual error indicators (red borders) and updates dynamic counter: *"Current Total: [X] / 100 points"*.
5. **System** displays modal alert: *"Validation Error: Total allocated bidding points must equal exactly 100 credits before locking preferences."*
6. **Student** adjusts points across selected electives to satisfy $\sum = 100$.
7. **System** re-enables submission button upon client-side formula match.

---

### Exception EF-02: Unmet Course Prerequisite
* **Associated Use Case:** UC-01 (Submit & Manage Elective Bids) / UC-02 (Validate Course Prerequisites)
* **Primary Actor:** Student
* **Trigger Condition:** Included use case `UC-02` queries the Academic Transcript Database and finds student lacks completed credit for required prerequisite course $P$ for elective $E$.

#### Step-by-Step Flow:
1. **System** executes prerequisite verification logic during bid pre-check.
2. **System** identifies that Student ID has not earned passing grade ($>D$) in prerequisite $P$.
3. **System** marks elective line item $E$ as *"Ineligible: Missing Prerequisite [P]"*.
4. **System** prevents locking of bid points on course $E$.
5. **System** displays remediation banner: *"Course [E] cannot be selected. Required prerequisite [P] not found in transcript records. Please select an eligible course or apply for a Registrar Waiver."*
6. **Student** replaces elective $E$ with an eligible course or contacts Registrar for waiver approval.

---

### Exception EF-03: Unresolvable Timetable Hard Collision
* **Associated Use Case:** UC-01 (Submit & Manage Elective Bids) / UC-03 (Resolve Timetable Conflict)
* **Primary Actor:** Student
* **Trigger Condition:** Extended use case `UC-03` detects that all top-ranked elective choices overlap with mandatory core department slots or with each other without alternate non-conflicting options.

#### Step-by-Step Flow:
1. **System** performs collision check across student's mandatory core timetable matrix.
2. **System** detects slot clash between Elective $A$ (Slot Mon 9:00 AM) and Core Course $C1$ (Slot Mon 9:00 AM), with zero secondary non-clashing electives selected.
3. **System** flags hard timetable violation.
4. **System** displays alert: *"Schedule Collision Error: Elective [A] conflicts with mandatory core course [C1]. Please adjust your elective ranking or choose courses from non-conflicting time slots."*
5. **Student** removes clashing course $A$ and selects an elective from an open timetable bucket.

---

### Exception EF-04: Bidding Window Expiry Lockout
* **Associated Use Case:** UC-01 (Submit & Manage Elective Bids)
* **Primary Actor:** Student
* **Trigger Condition:** Server clock timestamp $T_{\text{submit}} > T_{\text{deadline}}$ when student submits bid transaction.

#### Step-by-Step Flow:
1. **System** receives incoming HTTP POST payload for bid lock.
2. **System** checks active window status against NTP server clock.
3. **System** detects window closed at $T_{\text{deadline}}$.
4. **System** aborts payload write, revokes active draft write-lock, and marks student interface state as `READ_ONLY`.
5. **System** displays notice: *"Bidding Window Closed: The official deadline ([Timestamp]) has passed. No further bid submissions or modifications are accepted."*
6. **System** preserves last valid submission committed prior to $T_{\text{deadline}}$ as final student record.

---

### Exception EF-05: Optimization Solver Convergence / Timeout Failure
* **Associated Use Case:** UC-05 (Execute Elective Allocation Algorithm)
* **Primary Actor:** Allocation Engine / Academic Registrar
* **Trigger Condition:** Solver execution time exceeds 30-second performance constraint (NFR-001) or enters infeasible constraint matrix loop.

#### Step-by-Step Flow:
1. **Allocation Engine** initiates integer linear programming (ILP) matrix solver.
2. **Watchdog Timer** detects process runtime $> 30.0$ seconds without convergence.
3. **System** issues `SIGTERM` to solver worker process to prevent server memory exhaustion.
4. **System** initiates state rollback to pre-execution checkpoint (zero partial allocations committed).
5. **System** generates high-priority administrator alert: *"Solver Timeout Exception: Matrix allocation exceeded 30s threshold. System state rolled back safely."*
6. **Academic Registrar** reviews constraint parameters, relaxes soft capacity bounds, and triggers solver execution with updated configuration.

---

### Exception EF-06: Database Concurrency Deadlock under Peak Load
* **Associated Use Case:** UC-01 (Submit Elective Bids) / UC-05 (Execute Allocation Algorithm)
* **Primary Actor:** Student / System
* **Trigger Condition:** High volume of concurrent student submissions (5,000 active users per NFR-001) causes row lock contention or database connection pool starvation.

#### Step-by-Step Flow:
1. **Database Engine** encounters deadlock exception (`SQLSTATE 40P01`) on allocation table lock.
2. **System Middleware** catches exception and prevents raw error exposure to client.
3. **System** triggers automatic idempotent retry mechanism with exponential backoff (up to 3 retries).
4. If retries fail, **System** displays user notification: *"System is experiencing high load. Your bid request has been queued. Please wait a moment while we process your request."*
5. **System** writes transaction to message queue (RabbitMQ/Redis) for asynchronous processing, ensuring zero data loss.

---

### Exception EF-07: Unauthorized Access & Administrative Audit Alert
* **Associated Use Case:** UC-07 (Apply Manual Allocation Override)
* **Primary Actor:** Unauthorized User / Malicious Actor
* **Trigger Condition:** Unauthenticated session or non-registrar user attempts to call manual override API endpoint or tamper with bid logs (violating RBAC & NFR-002).

#### Step-by-Step Flow:
1. **RBAC Guard** intercepts request to `/api/v1/registrar/override`.
2. **System** verifies user JWT token and finds missing `ROLE_REGISTRAR` permission.
3. **System** rejects request immediately with HTTP 403 Forbidden status.
4. **System** appends immutable security alert entry to audit log containing IP address, user ID, timestamp, target resource, and security violation flag.
5. **System** locks IP address if repeated unauthorized attempts detected (> 5 failed calls).

---

## 4. System Resilience & Rollback Matrix

| Exception | Recovery Strategy | Data Integrity Action | User / Admin Action Required |
| :--- | :--- | :--- | :--- |
| **EF-01** | Client/Server Form Hold | Uncommitted; draft preserved locally. | Student adjusts point totals to 100. |
| **EF-02** | Line-Item Rejection | Uncommitted for ineligible course. | Student picks eligible course / requests waiver. |
| **EF-03** | Conflict Interception | Uncommitted; prior state retained. | Student selects alternate non-clashing slot. |
| **EF-04** | Atomic Window Seal | Transaction rolled back; read-only lock. | View final locked pre-deadline submission. |
| **EF-05** | Watchdog Execution Kill | Full DB Transaction Rollback to checkpoint. | Registrar adjusts solver soft constraints & re-runs. |
| **EF-06** | Idempotent Retry & Queue | Eventual Consistency via Queue. | None (automatic retry) or short wait. |
| **EF-07** | Session Terminate & Ban | Audit Log Locked (Write-Once). | System Security Admin notified. |
