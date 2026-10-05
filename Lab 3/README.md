# Lab 3: System Architecture and Design Justification
Course: PES University - Department of Computer Science and Engineering  
Student Name: Shashank D  
SRN: PES1UG24CS433

---

## 1. Overview

This folder contains the third laboratory deliverable for the Campus & Academic Operations problem domain. It builds directly on the work completed in Lab 1, where the problem was analyzed, requirements were gathered, and UML use-case models were created.

Lab 3 focuses on transforming the functional and non-functional requirements into a practical software architecture. The goal is to move from “what the system must do” to “how the system should be structured and why this design is suitable.”

The architecture and justification in this folder are based on the following problem:

> Academic Elective Bidding & Allocation System

---

## 2. Context from Lab 1

In Lab 1, the system was defined as a university-level academic operations platform that automates elective course selection and allocation.

### Problem Statement
The university needs an automated system for elective allocation where students submit preferences and bidding credits, and the system allocates courses based on constraints such as:

- student preference ranking
- bidding points
- CGPA tie-breakers
- prerequisite completion
- timetable conflict avoidance
- course capacity limits
- fairness and transparency

### Main Actors in Lab 1
1. Student
   - submits elective preferences
   - distributes bidding credits
   - views final allocation
   - handles resubmission if allowed

2. Academic Registrar
   - configures elective offerings
   - manages course capacity and sections
   - initiates allocation process
   - handles overrides if required

3. Allocation Engine
   - executes optimization logic
   - resolves conflicts
   - finalizes course assignment

4. Academic/Institutional Data Layer
   - prerequisite records
   - student transcript data
   - course catalog
   - timetable information

### Key Lab 1 Requirements
From the requirement engineering phase, the system was expected to:

- allow students to submit elective bids
- validate prerequisites before final submission
- support exactly 100 bidding credits per student
- avoid timetable clashes
- allocate electives using optimization rules
- maintain fairness using CGPA and bid strength
- ensure system performance under high load
- protect data and enforce role-based access control

### Lab 1 Deliverables
Lab 1 focused on:
- Requirements table
- Use case diagram
- Use case flow specifications
- Alternate flow specifications
- Exception flows
- System behavior under invalid or constrained conditions

These requirements form the foundation for the architectural decisions in Lab 3.

---

## 3. Problem Statement for This Lab

Lab 3 addresses the next step in the software engineering lifecycle:

How should the system be architecturally designed so that it can satisfy the requirements discovered in Lab 1?

This requires the system to be designed in a way that supports:

- student interaction
- registrar administration
- secure access control
- strong data integrity
- efficient optimization logic
- handling failures and exceptions gracefully
- scalability during peak submission periods

---

## 4. Goal of Lab 3

The objective of Lab 3 is to design and justify a suitable system architecture for the elective bidding and allocation workflow.

This includes:

- understanding the functional requirements from Lab 1
- identifying the major modules and components
- mapping those components to architectural layers
- deciding how data, logic, and users interact
- justifying the design using quality attributes such as:
  - scalability
  - maintainability
  - reliability
  - security
  - performance
  - modularity

---

## 5. Architectural Context

The architecture for this system is best modeled as a layered, service-based academic operations system.

### High-Level View
The platform can be visualized as a system with the following major parts:

1. Student Interface Layer
   - portal where students submit elective bids
   - allows course browsing
   - shows prerequisite warnings
   - displays final allocation results

2. Registrar/Admin Interface
   - manages course offerings
   - configures timetable slots and capacities
   - triggers optimization process
   - reviews allocation outcomes

3. Business Logic Layer
   - prerequisite validation
   - bidding logic
   - allocation engine
   - clash detection logic
   - timetable validation

4. Data Management Layer
   - student records
   - transcript information
   - electives and sections
   - bid submissions
   - audit logs

5. Security and Access Control
   - student authentication
   - registrar authorization
   - secure API access
   - immutable audit logs

6. Optimization / Decision Engine
   - computes final elective assignments
   - uses bid preference, GPA, capacity, and conflict rules
   - resolves allocation fairly

---

## 6. Why This Architecture Fits the Problem

This design is justified because the requirements from Lab 1 involve multiple kinds of responsibilities that must be separated cleanly.

### a) Separation of Concerns
The system has independent concerns:
- student interaction
- registrar configuration
- allocation computation
- validation rules
- database persistence
- security enforcement

This separation improves maintainability and reduces complexity.

### b) Security and Role-Based Access
The system must restrict administrative actions to authorized users only. The architecture therefore includes RBAC and audit tracking, which are essential for:
- registrar override actions
- course configuration
- final allocation decisions
- secure access to student records

### c) Performance and Scalability
During bidding windows, many students may submit simultaneously. Therefore, the system must support concurrency and fast processing:
- efficient validation checks
- batch submission processing
- optimized allocation algorithm
- database safeguards to avoid deadlocks

### d) Reliability and Fault Tolerance
The system must handle exceptions such as:
- invalid credit sums
- unmet prerequisites
- timetable conflicts
- solver timeouts
- lock contention
- late submissions

The architecture must therefore support rollback, validation checkpoints, and graceful failure handling.

### e) Expandability
Future academic operations may include:
- more departments
- additional elective types
- waitlist automation
- reporting dashboards
- mobile access
- integration with central university portals

A modular architecture makes it easier to extend these features later.

---

## 7. Major Components in the Architecture

### 1. Student Portal
Responsible for:
- login
- elective selection
- credit allocation
- bid review
- final result display

### 2. Course Catalog Service
Contains:
- elective offerings
- course descriptions
- instructor information
- timetable and section capacity

### 3. Prerequisite Validation Module
Checks:
- whether students meet prerequisites
- whether course restrictions apply
- whether students are eligible for a selected elective

### 4. Bidding Management Module
Handles:
- validation of total points
- bid storage
- draft and final submission logic
- closure and lock rules

### 5. Allocation Engine
Computes assignments based on:
- rank preferences
- bidding weight
- capacity constraints
- timetable clashes
- GPA tie-breakers

### 6. Timetable Conflict Resolver
Prevents:
- overlapping elective slots
- conflicting core and elective schedules
- unrealistic or impossible course combinations

### 7. Registrar Administration Module
Allows:
- course offering management
- seat capacity editing
- schedule updates
- allocation trigger
- override permissions

### 8. Audit and Security Layer
Stores:
- bid transaction history
- system events
- security alerts
- manual override activity
- user access operations

### 9. Database Layer
Stores all persistent information, such as:
- student data
- transcripts
- courses
- elective capacities
- bidding results
- allocation records

---

## 8. Architectural Diagram

This folder contains the architecture diagram file:

- `Architecture Diagram.png`

This diagram visually represents the interaction between core system components and their flow of data. It connects the user-facing processes with the academic data and allocation engine.

---

## 9. Deliverables in This Folder

This Lab 3 folder contains:

- `Architecture Diagram.png`  
  Visual architectural representation of the system

- `PES1UG24CS433_Lab3_Justification.pdf`  
  Document explaining the design decisions and justification for the architecture

---

## 10. Relationship Between Lab 1 and Lab 3

Lab 1 asked:

- what the system should do
- who the stakeholders are
- what requirements exist
- what exceptions and edge cases matter

Lab 3 asks:

- how should the system be structured to satisfy those requirements
- which modules are necessary
- how do the components interact
- why is this architecture the most appropriate

In short:

Lab 1 = Requirements Engineering  
Lab 3 = System Architecture Design

---

## 11. Conclusion

Lab 3 is the bridge between requirements and implementation. It takes the functional and non-functional expectations created in Lab 1 and organises them into a meaningful architecture for the Academic Elective Bidding & Allocation System.

The architecture is designed to be:
- modular
- secure
- scalable
- fair
- reliable
- easy to maintain

This makes it suitable for a real university environment where many students compete for limited elective seats under strict constraints.

---

## 12. Suggested Summary for Submission

“The Academic Elective Bidding & Allocation System is designed as a layered academic platform that supports student bidding, registrar configuration, constraint validation, and automated allocation. Based on the requirements established in Lab 1, the architecture separates user interfaces, business logic, scheduling rules, optimization services, security, and persistent data management. This ensures fairness, scalability, security, and reliability while handling prerequisite checks, timetable conflicts, and high-concurrency bidding periods.”

---

If you want, I can also generate:
1. a more formal academic README in report style,
2. a shorter student-friendly README,
3. or a version tailored exactly to your university submission format.
