# CrisisSync – Assignment 1, Part B: Requirements, Traceability and Acceptance Tests

**Group number:** 13

| Name                   | College e-mail                      |
|----------------------- |-------------------------------------|
| Lior Shor              | Lior.Shor@e.braude.ac.il            |
| Guy Gabi Kikozashvili  | Guy.Gabi.Kikozashvil@e.braude.ac.il |
| Amir Kosover           | Amir.Kosover@e.braude.ac.il         |
| Osher Amsalem          | Osher.Amsalem@e.braude.ac.il        |

---

## 1. Scope

This document defines requirements and acceptance tests for two operations of CrisisSync:

1. **Reserving a resource package for a task, including reporting task completion.**
2. **Generating the SLA (on-time performance) report.**

Page references (p.N) refer to the project story. References *Q&A-N* refer to the course staff's published clarifications (questions 1–6).

### 1.1 Assumptions

- **A-1 (Q&A-4):** An event is included in the SLA report of month *M* only if it was finally closed (after settlement approval) during month *M* and has at least one task in the **Completed** state. Its handling time is measured from the event opening until the completion time of its last **Completed** task; **Cancelled** tasks are ignored. Cancelled events and events that are not finally closed are excluded.
- **A-2:** Numeric thresholds in the NFRs were set by the team, since the story does not specify them. They shall be verified on the lab LAN with one server and up to 10 client stations.

---

## 2. Functional Requirements (FR)

### 2.1 Resource package reservation and task completion

| ID | Requirement | Source |
|----|-------------|--------|
| **FR-01** | The system shall allow the Dispatcher to reserve the resource package of a task only when the task is in the **Ready** state. | p.2 |
| **FR-02** | The system shall reserve the resource package as a single all-or-nothing operation: if any resource in the package is unavailable, the system shall reserve none of the package resources, keep the task in the **Ready** state, and notify the Dispatcher which resources were unavailable. | p.2 |
| **FR-03** | Upon a successful reservation, the system shall set the status of every package resource to **Reserved** and move the task to the **Active** state. | p.2 |
| **FR-04** | The system shall allow the Dispatcher to request automatic allocation of a task's resource package by a selected criterion (e.g., nearest available resources to the event location). If one or more selected resources are taken by a concurrent allocation, the system shall automatically replace only the taken resources with the next-best available alternatives, keep the rest of the selection, and retry the whole package as a new all-or-nothing transaction, with up to 2 alternative attempts (3 attempts in total), without the Dispatcher repeating the request. | p.3, Q&A-3 |
| **FR-05** | The system shall allow only the Field Commander currently assigned to a task to report the task's completion. When the assigned Field Commander is replaced, the permissions on the task shall pass to the replacement Field Commander. | p.1, p.3, Q&A-5 |
| **FR-06** | Upon a completion report, the system shall move the task to the **Completed** state, return all its resources to **Available**, and move to **Ready** every dependent task whose prerequisite tasks are all completed. | p.2, p.3 |
| **FR-07** | The system shall record every reservation, resource status change and task state transition in the audit log, together with the exact time and the user who performed the action. | p.3 |
| **FR-12** | When two or more Dispatchers concurrently request reservations that include the same resource, the system shall allow exactly one of the reservations to succeed; every other request shall fail as a whole without reserving any of its package resources, and the system shall notify the failing Dispatcher and refresh their screen with the current resource statuses. | p.2 |
| **FR-13** | The system shall fail an automatic allocation, reserving no resource and keeping the task in the **Ready** state, if (a) some package item has no available alternative in the pool, or (b) the second alternative attempt also fails due to concurrent reservations. The message to the Dispatcher shall distinguish between case (a), naming the item with no alternative, and case (b). | p.3, Q&A-3 |
| **FR-14** | The system shall reject a completion report for a task that is missing a package resource (released due to malfunction or unavailability and not yet replaced), unless the Dispatcher has confirmed that the missing resource is no longer required to complete the task. | Q&A-1, Q&A-5 |
| **FR-15** | When a reserved resource of an **Active** task is reported faulty or marked unavailable, the system shall immediately release it from the task, close its assignment record with an end time, keep the task in the **Active** state, and alert the Dispatcher to assign a replacement. | p.1, Q&A-1, Q&A-5 |

### 2.2 SLA report

| ID | Requirement | Source |
|----|-------------|--------|
| **FR-08** | The system shall automatically generate the SLA report at the end of each calendar month, according to the system clock. | p.4, p.5 |
| **FR-09** | The SLA report shall display, for each event type, the average time from event opening until the completion time of its last **Completed** task, over the events finally closed in the report month that have at least one **Completed** task; **Cancelled** tasks and cancelled events shall be ignored (see A-1). | p.4, Q&A-4 |
| **FR-10** | The system shall allow the System Administrator to generate the SLA report on demand for any selected past month. | p.5 |
| **FR-11** | The system shall allow only the System Administrator to view SLA reports. | p.1, p.4 |

---

## 3. Non-Functional Requirements (NFR)

### 3.1 Operation 1 – Resource package reservation and task completion

| ID | Category | Requirement | Source |
|----|----------|-------------|--------|
| *NFR-01* | Reliability | A reservation or completion that the server has confirmed to the client shall survive a server crash: after a server restart, *100%* of confirmed reservations and completions shall be present in the database, and no package shall be partially reserved. Client stations shall reconnect automatically and reload the current state within *10 seconds* after the server is back online. | p.2, p.4 |
| *NFR-02* | Performance | A resource package reservation (manual or automatic, including up to 2 fallback attempts) shall complete within *2 seconds* for at least 95% of requests, with 10 client stations working concurrently. | p.3, p.4 |
| *NFR-03* | Performance | A change in a task state or resource status shall be pushed by the server (no client polling) and displayed on all relevant open client stations within *1 second*. | p.3 |
| *NFR-04* | Security | A session with no user activity for **15 minutes** shall be closed automatically and require a new login. After **5** consecutive failed login attempts, the account shall be locked for **15 minutes**. | p.1 |
| *NFR-05* | Usability | A trained Dispatcher shall complete a manual reservation of a 4-resource package, starting from the event status screen, within *30 seconds* on average over 10 trials, and an automatic allocation request in no more than *3* user actions. | p.4 |
| *NFR-11* | Reliability | Every operation that changes an event, its tasks or reservations (reservation, completion report, malfunction report, task cancellation) shall acquire locks in one fixed order – event record, then task records, then resource records by ascending ID – so that in *100* concurrent trials of conflicting operations on the same event, *0* deadlocks occur and every operation either completes or fails cleanly within *5 seconds*. | p.2, Q&A-6 |

### 3.2 Operation 2 – SLA report

| ID | Category | Requirement | Source |
|----|----------|-------------|--------|
| *NFR-06* | Performance | Generating a monthly SLA report over up to *5,000* events shall complete within *10 seconds*. | p.4 |
| *NFR-07* | Reliability | If the server is down when the month ends, the missed automatic SLA report shall be generated within *5 minutes* after the server restarts. A generated report shall never be lost or partially stored (100% of runs). | p.4, p.5 |
| *NFR-08* | Security | Generated SLA reports shall be stored read-only: **100%** of attempts to modify or delete a stored report through the client application shall be rejected. | p.4 |
| *NFR-09* | Usability | The System Administrator shall open the SLA report of any past month in no more than *3* user actions from the main screen, and average times shall be displayed in a uniform *hh:mm* format. | p.4, p.5 |

### 3.3 Both operations

| ID | Category | Requirement | Source |
|----|----------|-------------|--------|
| *NFR-10* | Security | User passwords shall be stored only as *bcrypt* hashes with a cost factor of at least *10*; plain-text passwords shall never be stored or logged. | p.1 |

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
| UC-07 | Log In (precondition of UC-01, UC-03, UC-05, UC-06) | User (generalization of all human actors) |
| UC-23 | Assign Replacement Resource | Dispatcher |
| UC-31 | Report Equipment Malfunction | Field Commander, Field Crew Member |

### 4.2 Matrix

| Req ID | Short description | Source | Use Case | Acceptance Test(s) |
|--------|-------------------|--------|----------|--------------------|
| FR-01 | Reserve only in Ready state | p.2 | UC-01 | AT-01, AT-02 |
| FR-02 | All-or-nothing reservation | p.2 | UC-01 | AT-03 |
| FR-03 | Resources Reserved, task Active | p.2 | UC-01 | AT-01 |
| FR-04 | Automatic allocation, replace only taken resources, 3 attempts total | p.3, Q&A-3 | UC-02 | AT-04 |
| FR-05 | Only the currently assigned Field Commander completes | p.1, p.3, Q&A-5 | UC-03 | AT-06, AT-07, AT-26 |
| FR-06 | Completion releases resources and promotes dependents | p.2, p.3 | UC-03 | AT-06 |
| FR-07 | Audit log of reservation and completion | p.3 | UC-01, UC-03 | AT-01, AT-06 |
| FR-08 | Automatic monthly SLA report | p.4, p.5 | UC-04 | AT-08 |
| FR-09 | Average time to last Completed task, per event type, closed events only | p.4, Q&A-4 | UC-04, UC-05 | AT-08, AT-28 |
| FR-10 | On-demand report for a past month | p.5 | UC-05 | AT-09, AT-10 |
| FR-11 | Only Administrator views reports | p.1, p.4 | UC-06 | AT-11 |
| FR-12 | Concurrent reservation, exactly one succeeds | p.2 | UC-01, UC-02 | AT-12 |
| FR-13 | Two distinguished automatic-allocation failure cases | p.3, Q&A-3 | UC-02 | AT-05, AT-23 |
| FR-14 | No completion while a resource is missing, unless waived | Q&A-1, Q&A-5 | UC-03 | AT-24 |
| FR-15 | Faulty/unavailable resource released at once, task stays Active | p.1, Q&A-1, Q&A-5 | UC-31, UC-23 | AT-25 |
| NFR-01 | Confirmed operations survive server crash, reconnect ≤ 10 s | p.2, p.4 | UC-01, UC-03 | AT-18 |
| NFR-02 | Reservation ≤ 2 s (95%) | p.3, p.4 | UC-01, UC-02 | AT-13 |
| NFR-03 | Push update ≤ 1 s | p.3 | UC-01, UC-03 | AT-14 |
| NFR-04 | 15-min session timeout, lockout after 5 failed logins | p.1 | UC-07, UC-01, UC-03 | AT-19 |
| NFR-05 | Manual reservation ≤ 30 s, auto ≤ 3 actions | p.4 | UC-01, UC-02 | AT-16 |
| NFR-06 | SLA report ≤ 10 s for 5,000 events | p.4 | UC-04, UC-05 | AT-17 |
| NFR-07 | Missed monthly report generated ≤ 5 min after restart | p.4, p.5 | UC-04 | AT-20 |
| NFR-08 | Stored reports are read-only | p.4 | UC-06 | AT-21 |
| NFR-09 | Open report ≤ 3 actions, hh:mm format | p.4, p.5 | UC-06 | AT-22 |
| NFR-10 | bcrypt hashes, cost ≥ 10 | p.1 | UC-07 | AT-15 |
| NFR-11 | Fixed lock order, 0 deadlocks in 100 trials | p.2, Q&A-6 | UC-01, UC-03, UC-31 | AT-27 |

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
Given events in Haifa and Tel Aviv each have a Ready task that needs a bulldozer and a paramedic
And bulldozers B1 and B2 are Available and B1 is the nearest to both events
And paramedic P1 is the nearest paramedic to Haifa and P2 is the nearest paramedic to Tel Aviv
When both Dispatchers request automatic allocation by "nearest location" at the same moment
Then exactly one task receives B1
And the other task automatically receives B2 in a new all-or-nothing attempt without the Dispatcher repeating the request
And that task keeps its originally selected paramedic
And both tasks move to the Active state
```

**AT-05 – Automatic allocation fails when an item has no alternative (error)** — FR-13
```gherkin
Given a Ready task needs a bulldozer and a paramedic
And every bulldozer in the pool is Reserved or Faulty/In Maintenance
When the Dispatcher requests automatic allocation
Then the allocation fails and no resource is reserved
And the task remains in the Ready state
And the Dispatcher is notified "No available alternative for: Bulldozer"
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

**AT-07 – Completion by a non-assigned Field Commander is rejected (error)** — FR-05
```gherkin
Given task "Debris Removal" is Active with Field Commander "FC-A" assigned
And Field Commander "FC-B" is logged in
When "FC-B" tries to report completion of "Debris Removal"
Then the system rejects the request with an authorization error
And the task remains Active and its resources remain Reserved
```

**AT-23 – Automatic allocation fails after 3 contended attempts (error)** — FR-13
```gherkin
Given a Ready task needs a bulldozer
And bulldozers B1, B2 and B3 are Available
And each time the task's allocation selects a bulldozer, a concurrent allocation reserves it first
When the Dispatcher requests automatic allocation
Then the system makes exactly 3 attempts (B1, then B2, then B3)
And the allocation fails and no resource is reserved for the task
And the task remains in the Ready state
And the Dispatcher is notified "Allocation failed due to concurrent reservations – please try again"
```

**AT-24 – Completion blocked while a resource is missing (error and recovery)** — FR-14
```gherkin
Given task "Debris Removal" is Active
And its bulldozer was reported faulty and released, and no replacement was assigned
When the assigned Field Commander reports completion of "Debris Removal"
Then the system rejects the report with the message "Task is missing a required resource"
And the task remains Active
When the Dispatcher confirms that the bulldozer is no longer required for the task
And the Field Commander reports completion again
Then the task moves to the Completed state
```

**AT-25 – Malfunction during an Active task (error and recovery)** — FR-15
```gherkin
Given task "Debris Removal" is Active with bulldozer B1 reserved since 10:00
When a Field Crew Member assigned to the task reports B1 as faulty at 12:30
Then B1 is set to Faulty/In Maintenance and is no longer reserved for the task
And the assignment record of B1 is closed with end time 12:30
And the task remains in the Active state
And the Dispatcher receives an alert to assign a replacement bulldozer
```

**AT-26 – Replaced Field Commander loses completion permission (security)** — FR-05
```gherkin
Given task "Debris Removal" is Active with Field Commander "FC-A" assigned
And "FC-A" was marked unavailable and the Dispatcher assigned "FC-B" as the replacement
When "FC-A" tries to report completion of "Debris Removal"
Then the system rejects the request with an authorization error
When "FC-B" reports completion of "Debris Removal"
Then the task moves to the Completed state
```

### Operation 2 – SLA report

**AT-08 – Automatic monthly SLA report (happy path)** — FR-08, FR-09
```gherkin
Given during October two "Fire" events were finally closed, whose last Completed tasks were completed 2 h and 4 h after the events were opened
And one "Road Accident" event was finally closed in October, whose last Completed task was completed 1 h after it was opened
When the system clock reaches the end of October
Then the system generates the October SLA report automatically
And the report shows an average of 3 h for "Fire" and 1 h for "Road Accident"
```

**AT-09 – On-demand report for a past month (happy path)** — FR-10
```gherkin
Given the System Administrator is logged in
And some events were finally closed during August
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

**AT-11 – Report access is restricted to the Administrator (security)** — FR-11
```gherkin
Given a Dispatcher is logged in
When the Dispatcher tries to open an SLA report
Then the system denies access
And no report data is sent to the Dispatcher's station
```

**AT-28 – SLA report includes only closed events with Completed tasks (edge cases)** — FR-09
```gherkin
Given the following "Fire" events exist:
  | Event | Status in October                | Tasks                                                    |
  | E1    | Finally closed on Oct 10         | last Completed 3 h after opening, a later task Cancelled |
  | E2    | Finally closed on Oct 12         | all tasks Cancelled                                      |
  | E3    | Cancelled on Oct 15              | one task Completed 1 h after opening                     |
  | E4    | Finally closed on Oct 2          | last Completed on Sep 30, 5 h after opening              |
  | E5    | Open (settlement not approved)   | all tasks Completed                                      |
When the October SLA report is generated
Then the "Fire" average is 4 h, calculated from E1 (3 h) and E4 (5 h) only
And the Cancelled task of E1 does not extend its handling time
And E2, E3 and E5 are not included in the report
```
**AT-12 – Concurrent reservation of the same resource (error and recovery)** — FR-12
```gherkin
Given bulldozer B1 is Available
And two Dispatcher stations each have a Ready task whose package includes B1
When both Dispatchers reserve their packages at the same moment, repeated 100 times
Then in every trial exactly one reservation succeeds
And the failed reservation leaves none of its package resources Reserved
And the failing Dispatcher receives a clear message and an updated screen
```

### Non-functional tests



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

**AT-15 – Passwords are stored hashed (security)** — NFR-10
```gherkin
Given a new user account is created with a password
When the users table is inspected in the database
Then the stored value is a bcrypt hash with cost factor of at least 10
And the plain-text password does not appear in the database
```

**AT-16 – Reservation efficiency (usability)** — NFR-05
```gherkin
Given a trained Dispatcher is on the status screen of an open event with a Ready task
And the task's package contains 4 resources
When the Dispatcher performs a manual reservation 10 times
Then the average completion time is at most 30 seconds
When the Dispatcher requests automatic allocation for a Ready task
Then the request is submitted in no more than 3 user actions
```

**AT-17 – SLA report generation time (performance)** — NFR-06
```gherkin
Given the database contains 5,000 events that were finally closed in the selected month
When the System Administrator generates the SLA report for that month
Then the report is displayed within 10 seconds
```
**AT-18 – Confirmed reservation survives server crash (reliability)** — NFR-01
```gherkin
Given the Dispatcher received a confirmation for reserving the package of "Debris Removal"
When the server process is killed and restarted
Then all resources of "Debris Removal" are still Reserved and the task is Active
And no package in the database is partially reserved
And every connected client station reconnects and shows the current state within 10 seconds
```
**AT-19 – Session timeout and login lockout (security)** — NFR-04
```gherkin
Given a Dispatcher is logged in
When the Dispatcher performs no action for 15 minutes
Then the session is closed and the next action requires a new login
Given a user enters a wrong password 5 times in a row
When the user tries to log in again with the correct password within 15 minutes
Then the login is rejected with the message "Account locked – try again later"
And after 15 minutes the user can log in successfully
```
**AT-20 – Missed monthly report is recovered (reliability)** — NFR-07
```gherkin
Given the server is down at the end of October
When the server restarts on November 1st
Then the October SLA report is generated within 5 minutes after the restart
```
**AT-21 – Stored report is read-only (security)** — NFR-08
```gherkin
Given the October SLA report has been generated
When the System Administrator opens the report
Then no edit or delete action is available
When a modify or delete request for the stored report is sent from the client
Then the server rejects the request and the stored report values remain unchanged
```
**AT-22 – Report is quick to open (usability)** — NFR-09
```gherkin
Given the System Administrator is on the main screen
When the Administrator opens the SLA report of a past month
Then the report is displayed after no more than 3 user actions
And all average times are shown in hh:mm format
```
**AT-27 – No deadlock between conflicting operations (reliability)** — NFR-11
```gherkin
Given task "Debris Removal" is Active with bulldozer B1 reserved
And task "Fire Suppression" in the same event is Ready and its package includes paramedic P1
When, at the same moment, a Field Crew Member reports B1 as faulty, the Field Commander reports completion of "Debris Removal", and a Dispatcher reserves the package of "Fire Suppression", repeated 100 times
Then no deadlock occurs in any trial
And every operation either completes or fails with a clear message within 5 seconds
And the operation that arrived second is validated against the updated state
```