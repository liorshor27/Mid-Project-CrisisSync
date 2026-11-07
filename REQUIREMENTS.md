# CrisisSync – Assignment 1, Part B: Requirements, Traceability and Acceptance Tests

**Group number:** 13

| Name                  | ID        |College e-mail                       |
|-----------------------|-----------|-------------------------------------|
| Lior Shor             | 212272793 | Lior.Shor@e.braude.ac.il            |
| Guy Gabi Kikozashvili | 324096403 | Guy.Gabi.Kikozashvil@e.braude.ac.il |
| Amir Kosover          | 212131296 | Amir.Kosover@e.braude.ac.il         |
| Osher Amsalem         | 212393748 | Osher.Amsalem@e.braude.ac.il        |

---

## 1. Scope

This document defines requirements and acceptance tests for two operations of CrisisSync:

1. **Reserving a resource package for a task, including reporting task completion.**
2. **Generating the SLA (on-time performance) report.**

Page references (p.N) refer to the project story.

### 1.1 Assumptions

- **A-1:** An event is included in the SLA report of month *M* if the last of its tasks reached a final state (Completed or Cancelled) during month *M*. Events that still have a non-final task are excluded.
- **A-2:** Numeric thresholds in the NFRs were set by the team, since the story does not specify them. They shall be verified on the lab LAN with one server and up to 10 client stations.

---

## 2. Functional Requirements (FR)

### 2.1 Resource package reservation and task completion

| ID | Requirement | Source |
|----|-------------|--------|
| **FR-01** | The system shall allow the Dispatcher to reserve the resource package of a task only when the task is in the **Ready** state. | p.2 |
| **FR-02** | The system shall reserve the resource package as a single all-or-nothing operation: if any resource in the package is unavailable, the system shall reserve none of the package resources, keep the task in the **Ready** state, and notify the Dispatcher which resources were unavailable. | p.2 |
| **FR-03** | Upon a successful reservation, the system shall set the status of every package resource to **Reserved** and move the task to the **Active** state. | p.2 |
| **FR-04** | The system shall allow the Dispatcher to request automatic allocation of a task's resource package by a selected criterion (e.g., nearest available resources to the event location). If a selected resource is taken by a concurrent allocation, the system shall automatically try the next-best available alternative, up to 2 alternative attempts, and shall report failure only if no alternative is available. | p.3 |
| **FR-05** | The system shall allow only the Field Commander assigned to a task to report the task's completion. | p.1, p.3 |
| **FR-06** | Upon a completion report, the system shall move the task to the **Completed** state, return all its resources to **Available**, and move to **Ready** every dependent task whose prerequisite tasks are all completed. | p.2, p.3 |
| **FR-07** | The system shall record every reservation, resource status change and task state transition in the audit log, together with the exact time and the user who performed the action. | p.3 |

### 2.2 SLA report

| ID | Requirement | Source |
|----|-------------|--------|
| **FR-08** | The system shall automatically generate the SLA report at the end of each calendar month, according to the system clock. | p.4, p.5 |
| **FR-09** | The SLA report shall display, for each event type, the average time from event opening until the closure of its last task, for the events included in the report month (see A-1). | p.4 |
| **FR-10** | The system shall allow the System Administrator to generate the SLA report on demand for any selected past month. | p.5 |
| **FR-11** | The system shall allow only the System Administrator to view SLA reports. | p.1, p.4 |

---

## 3. Non-Functional Requirements (NFR)

| ID | Category | Requirement | Source |
|----|----------|-------------|--------|
| **NFR-01** | Reliability | When two Dispatchers attempt to reserve the same resource concurrently, exactly one reservation shall succeed in 100% of 100 repeated concurrent trials, and no partially reserved package shall remain in the database after a failed reservation (full rollback). | p.2, p.5 |
| **NFR-02** | Performance | A resource package reservation (manual or automatic, including up to 2 fallback attempts) shall complete within **2 seconds** for at least 95% of requests, with 10 client stations working concurrently. | p.3, p.4 |
| **NFR-03** | Performance | A change in a task state or resource status shall be pushed by the server (no client polling) and displayed on all relevant open client stations within **1 second**. | p.3 |
| **NFR-04** | Security | User passwords shall be stored only as **bcrypt** hashes with a cost factor of at least **10**. Any completion report or report-view request from a user without the required role shall be rejected in 100% of attempts. | p.1 |
| **NFR-05** | Usability | The Dispatcher shall be able to complete a manual reservation from the event status screen in no more than **3** user actions (clicks or key presses), and an error message after a failed reservation shall appear within **1 second**. | p.2, p.4 |
| **NFR-06** | Performance | Generating a monthly SLA report over up to **5,000** events shall complete within **10 seconds**. | p.4 |

---

## 4. Requirements Traceability Matrix (RTM)

### 4.1 Use Cases referenced

| UC ID | Use Case | Primary actor |
|-------|----------|---------------|
| UC-01 | Reserve Resource Package | Dispatcher |
| UC-02 | Auto-Allocate Resource Package (`<<extend>>` UC-01) | Dispatcher |
| UC-03 | Report Task Completion | Field Commander |
| UC-04 | Generate Monthly SLA Report | Scheduler (timer) |
| UC-05 | Generate SLA Report On Demand | System Administrator |
| UC-06 | View SLA Report | System Administrator |
| UC-07 | Log In (`<<include>>` from UC-01, UC-03, UC-05, UC-06) | All human users |

### 4.2 Matrix

| Req ID | Short description | Source | Use Case | Acceptance Test(s) |
|--------|-------------------|--------|----------|--------------------|
| FR-01 | Reserve only in Ready state | p.2 | UC-01 | AT-01, AT-02 |
| FR-02 | All-or-nothing reservation | p.2 | UC-01 | AT-03 |
| FR-03 | Resources Reserved, task Active | p.2 | UC-01 | AT-01 |
| FR-04 | Automatic allocation with up to 2 fallbacks | p.3 | UC-02 | AT-04, AT-05 |
| FR-05 | Only assigned Field Commander completes | p.1, p.3 | UC-03 | AT-06, AT-07 |
| FR-06 | Completion releases resources and promotes dependents | p.2, p.3 | UC-03 | AT-06 |
| FR-07 | Audit log of reservation and completion | p.3 | UC-01, UC-03 | AT-01, AT-06 |
| FR-08 | Automatic monthly SLA report | p.4, p.5 | UC-04 | AT-08 |
| FR-09 | Average time per event type | p.4 | UC-04, UC-05 | AT-08 |
| FR-10 | On-demand report for a past month | p.5 | UC-05 | AT-09, AT-10 |
| FR-11 | Only Administrator views reports | p.1, p.4 | UC-06 | AT-11 |
| NFR-01 | Concurrent reservation, exactly one succeeds | p.2, p.5 | UC-01 | AT-12 |
| NFR-02 | Reservation ≤ 2 s (95%) | p.3, p.4 | UC-01, UC-02 | AT-13 |
| NFR-03 | Push update ≤ 1 s | p.3 | UC-01, UC-03 | AT-14 |
| NFR-04 | bcrypt hashes, role enforcement | p.1 | UC-07, UC-03, UC-06 | AT-07, AT-11, AT-15 |
| NFR-05 | ≤ 3 actions, error ≤ 1 s | p.2, p.4 | UC-01 | AT-16 |
| NFR-06 | SLA report ≤ 10 s for 5,000 events | p.4 | UC-04, UC-05 | AT-17 |

---

## 5. Acceptance Tests (BDD – Given-When-Then)

### Operation 1 – Resource package reservation and task completion

**AT-01 – Successful manual reservation (happy path)** — FR-01, FR-03, FR-07
```gherkin
Given the Dispatcher is logged in
  And task "Debris Removal" is in the Ready state
  And its package (Field Commander, Heavy Equipment Operator, Bulldozer, Paramedic) is fully Available
When the Dispatcher reserves the package for "Debris Removal"
Then all four resources are set to Reserved
  And the task "Debris Removal" moves to the Active state
  And the audit log contains the reservation with the time and the Dispatcher's username
```

**AT-02 – Reservation rejected for a Blocked task (error)** — FR-01
```gherkin
Given task "Structural Shoring" is in the Blocked state with an open prerequisite task
When the Dispatcher tries to reserve its package
Then the system rejects the reservation with the message "Task is not Ready"
  And no resource changes its status
  And the task remains in the Blocked state
```

**AT-03 – One resource unavailable, full rollback (error and recovery)** — FR-02
```gherkin
Given task "Debris Removal" is in the Ready state
  And the bulldozer in its package has status Faulty/In Maintenance
When the Dispatcher reserves the package
Then the reservation fails and no resource in the package is Reserved
  And the task remains in the Ready state
  And the Dispatcher sees a message naming the bulldozer as unavailable
  And the Dispatcher's screen shows the current resource statuses
```

**AT-04 – Automatic allocation with fallback (happy path with conflict)** — FR-04
```gherkin
Given events in Haifa and Tel Aviv each have a Ready task that needs a bulldozer
  And bulldozers B1 and B2 are Available and B1 is the nearest to both events
When both Dispatchers request automatic allocation by "nearest location" at the same moment
Then exactly one task receives B1
  And the other task automatically receives B2 without the Dispatcher repeating the request
  And both tasks move to the Active state
```

**AT-05 – Automatic allocation fails when no alternative is left (error)** — FR-04
```gherkin
Given a Ready task needs a bulldozer
  And no bulldozer is Available after 2 alternative attempts
When the Dispatcher requests automatic allocation
Then the allocation fails and no resource is reserved
  And the task remains in the Ready state
  And the Dispatcher is notified that no available alternative exists
```

**AT-06 – Assigned Field Commander reports completion (happy path)** — FR-05, FR-06, FR-07
```gherkin
Given task "Debris Removal" is Active and its assigned Field Commander is logged in
  And task "Search and Rescue" depends only on "Debris Removal"
When the Field Commander reports completion of "Debris Removal"
Then "Debris Removal" moves to the Completed state
  And all its resources return to Available
  And "Search and Rescue" moves from Blocked to Ready
  And the audit log records the transition with the time and the Field Commander's username
```

**AT-07 – Completion by a non-assigned Field Commander is rejected (error)** — FR-05, NFR-04
```gherkin
Given task "Debris Removal" is Active with Field Commander "FC-A" assigned
  And Field Commander "FC-B" is logged in
When "FC-B" tries to report completion of "Debris Removal"
Then the system rejects the request with an authorization error
  And the task remains Active and its resources remain Reserved
```

### Operation 2 – SLA report

**AT-08 – Automatic monthly SLA report (happy path)** — FR-08, FR-09
```gherkin
Given during October two "Fire" events were closed after 2 h and 4 h
  And one "Road Accident" event was closed after 1 h
When the system clock reaches the end of October
Then the system generates the October SLA report automatically
  And the report shows an average of 3 h for "Fire" and 1 h for "Road Accident"
```

**AT-09 – On-demand report for a past month (happy path)** — FR-10
```gherkin
Given the System Administrator is logged in
  And events were closed during August
When the Administrator requests the SLA report for August
Then the system displays the August SLA report grouped by event type
```

**AT-10 – On-demand report for a future month or an empty month (error)** — FR-10
```gherkin
Given the System Administrator is logged in
When the Administrator requests the SLA report for a month that has not ended yet
Then the system rejects the request with the message "Select a past month"
When the Administrator requests a past month with no closed events
Then the system displays an empty report with the message "No closed events in this month"
```

**AT-11 – Report access is restricted to the Administrator (security)** — FR-11, NFR-04
```gherkin
Given a Dispatcher is logged in
When the Dispatcher tries to open an SLA report
Then the system denies access
  And no report data is sent to the Dispatcher's station
```

### Non-functional tests

**AT-12 – Concurrent reservation of the same resource (reliability)** — NFR-01
```gherkin
Given bulldozer B1 is Available
  And two Dispatcher stations each have a Ready task whose package includes B1
When both Dispatchers reserve their packages at the same moment, repeated 100 times
Then in every trial exactly one reservation succeeds
  And the failed reservation leaves none of its package resources Reserved
  And the failing Dispatcher receives a clear message and an updated screen
```

**AT-13 – Reservation response time (performance)** — NFR-02
```gherkin
Given 10 client stations are connected to the server
When 200 reservation requests are sent from the stations within 1 minute
Then at least 95% of the reservations complete within 2 seconds
```

**AT-14 – Real-time push of status changes (performance)** — NFR-03
```gherkin
Given three client stations show the status screen of the same event
When a task in that event changes from Active to Completed
Then all three stations show the new state within 1 second
  And no station sent a polling request for the change
```

**AT-15 – Passwords are stored hashed (security)** — NFR-04
```gherkin
Given a new user account is created with a password
When the users table is inspected in the database
Then the stored value is a bcrypt hash with cost factor of at least 10
  And the plain-text password does not appear in the database
```

**AT-16 – Reservation in few actions (usability)** — NFR-05
```gherkin
Given the Dispatcher is on the status screen of an open event with a Ready task
When the Dispatcher performs a manual reservation of the task's package
Then the reservation is completed in no more than 3 user actions
When a reservation fails
Then the error message appears within 1 second
```

**AT-17 – SLA report generation time (performance)** — NFR-06
```gherkin
Given the database contains 5,000 events closed in the selected month
When the System Administrator generates the SLA report for that month
Then the report is displayed within 10 seconds
```
