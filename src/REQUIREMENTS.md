# G2_Assignment1
**Course:** Software Engineering Methods  
**System:** CrisisSync

### Engineering Team Details
- **Student 1:** Amir Sanallah | **ID:** 212716773 | **Braude Email:** Amir.Sanallah@e.braude.ac.il
- **Student 2:** Kamal Hamdony | **ID:** 324833516 | **Braude Email:** Kamal.Hamdony@e.braude.ac.il
- **Student 3:** Mohamad Shalata | **ID:** 326564846 | **Braude Email:** Mohamad.Shalata@e.braude.ac.il
- **Student 4:** Bar Nahman | **ID:** 207851213 | **Braude Email:** Bar.Nahman@e.braude.ac.il

---

## Requirements Traceability Matrix (RTM)

### Operation 1: Resource Package Allocation and Task Completion Reporting

| Req ID | Requirement Description | Type | Mapped Use Case | Test ID |
| :--- | :--- | :---: | :--- | :---: |
| **REQ-01** | The system shall accept a task's resource package definition and allocation request only when the package contains **exactly one** Field Commander designated as responsible for that task, rejecting any package with zero or multiple Field Commanders. | FR | Create Task; Manually Allocate Resources; Automatically Allocate Resources | TEST-01, TEST-02, TEST-03 |
| **REQ-02** | The system shall allocate a complete resource package to a task in one atomic SQL transaction only when the task is in the `Ready` state, all its predecessors are `Completed`, and all requested resources are `Available`; if any resource in the package cannot be reserved, the system shall roll back the entire transaction and leave the task in the `Ready` state. | FR | Manually Allocate Resources; Automatically Allocate Resources; Verify & Lock Resource Package; Rollback Allocation on Resource Conflict | TEST-03, TEST-04, TEST-06 |
| **REQ-03** | During automatic resource allocation by a selected criterion, if a selected resource becomes unavailable due to a concurrent allocation, the system shall automatically attempt the next-best available alternative resource for up to **2 alternative attempts** without requiring Dispatcher re-submission; if no valid alternative is available, the system shall roll back the transaction with zero partial reservations. | FR | Automatically Allocate Resources; Fallback to Next Best Available Resource; Rollback Allocation on Resource Conflict | TEST-05, TEST-06 |
| **REQ-04** | The system shall allow only the designated Field Commander of an `Active` task to report its completion; upon completion, the system shall transition the task to `Completed`, immediately release all assigned operational resources to `Available` while preserving the `Broken` status of any faulty equipment, transition dependent tasks to `Ready` when all their predecessors are `Completed`, and record the status transition in the Audit Log. | FR | Report Task Completion | TEST-07, TEST-09 |
| **REQ-05** | The system shall complete (commit or roll back) any resource package allocation transaction within a maximum of **1.5 seconds**, measured from the server's receipt of the allocation request until transaction completion. | NFR | Manually Allocate Resources; Automatically Allocate Resources; Verify & Lock Resource Package | TEST-03, TEST-04, TEST-05, TEST-06 |
| **REQ-06** | The system shall push task status and resource availability updates via Server Push (without client polling) to all connected, authorized client screens subscribed to the affected event within a maximum of **1.0 second** of each successful state change. | NFR | Monitor Events in Real Time; View Assigned Events and Tasks | TEST-03, TEST-07, TEST-15 |
| **REQ-07** | The system shall visually highlight any `Active` task on live event status screens within a maximum of **1.0 second** after its elapsed execution time exceeds its defined estimated duration. | NFR | Monitor Events in Real Time; View Assigned Events and Tasks | TEST-08 |
| **REQ-08** | The system shall prevent double-booking in **100%** of concurrent resource allocation requests by enforcing transactional pessimistic locking during resource verification and reservation. | NFR | Manually Allocate Resources; Automatically Allocate Resources; Verify & Lock Resource Package | TEST-04, TEST-05, TEST-06 |
| **REQ-09** | The system shall reject **100%** of task-completion requests submitted by any user who is not the designated Field Commander for that specific task, leaving the task status and its reserved resources unchanged. | NFR | Report Task Completion | TEST-09 |
| **REQ-10** | The system shall allow a Dispatcher to initiate resource package allocation (manually or automatically) and a designated Field Commander to report task completion from their respective live event status screens in no more than **3 mouse clicks** per operation. | NFR | Manually Allocate Resources; Automatically Allocate Resources; Report Task Completion; Monitor Events in Real Time; View Assigned Events and Tasks | TEST-03, TEST-05, TEST-07 |

### Operation 2: Generating the SLA (Time Adherence) Report

| Req ID | Requirement Description | Type | Mapped Use Case | Test ID |
| :--- | :--- | :---: | :--- | :---: |
| **REQ-11** | The system shall automatically generate the monthly administrative reports (Resource Utilization and SLA Time Adherence reports) at the end of each calendar month and shall also allow the System Administrator to generate the SLA report on demand for any selected past month. | FR | Generate Reports; View Reports | TEST-10, TEST-11 |
| **REQ-12** | The system shall calculate and display in the SLA report, grouped by emergency event type, the average elapsed time from event opening to the completion of the final task for the events included in the selected reporting period. | FR | Generate Reports; View Reports | TEST-10 |
| **REQ-13** | The system shall generate and display an on-demand monthly SLA report within a maximum of **5.0 seconds** from the moment the System Administrator submits the request. | NFR | Generate Reports; View Reports | TEST-10, TEST-13 |
| **REQ-14** | If an SLA report query returns **0** eligible events or exceeds the **5.0-second** timeout threshold, the system shall display an explicit empty-state or timeout message within **0.5 seconds (500 ms)** of detection while keeping the client application usable to retry or select another reporting period. | NFR | Generate Reports; View Reports | TEST-12, TEST-13 |
| **REQ-15** | The system shall restrict interactive administrative report generation and report viewing exclusively to authenticated `System Administrator` accounts, rejecting **100%** of unauthorized access attempts by other user roles, while permitting the authorized `Monthly Report Scheduler` to trigger automatic end-of-month report generation. | NFR | Generate Reports; View Reports | TEST-11, TEST-14 |

---

## BDD Acceptance Tests

### TEST-01: Reject an invalid resource package with zero Field Commanders
**Given** a Dispatcher is creating a new task and defining its resource package  
**When** the Dispatcher submits a resource package containing **zero** Field Commanders  
**Then** the system shall reject the task creation and invalid resource package  
**And** the system shall not save the invalid package or reserve any resources.

---

### TEST-02: Reject an invalid resource package with multiple Field Commanders
**Given** a Dispatcher is creating a new task and defining its resource package  
**When** the Dispatcher submits a resource package containing **more than one** Field Commander  
**Then** the system shall reject the task creation and invalid resource package  
**And** the system shall not save the invalid package or reserve any resources.

---

### TEST-03: Successfully allocate a resource package manually
**Given** a task is in the `Ready` state, all its predecessor tasks are `Completed`, its resource package contains **exactly one** designated Field Commander, and all requested resources are `Available`  
**When** the Dispatcher manually allocates the resource package to the task from the live event status screen  
**Then** the system shall reserve all requested resources in one atomic SQL transaction and transition the task to `Active` within **1.5 seconds**  
**And** the system shall push the updated task and resource statuses to all connected, authorized screens viewing the event within **1.0 second**  
**And** the Dispatcher shall be able to initiate the allocation from the live status screen in no more than **3 mouse clicks**.

---

### TEST-04: Roll back a conflicting manual resource allocation atomically
**Given** two Dispatchers concurrently submit manual allocation requests for two different `Ready` tasks whose resource packages both require the same `Available` resource alongside other available resources  
**When** both allocation transactions attempt to lock and reserve the shared resource simultaneously  
**Then** the system shall enforce transactional pessimistic locking so that **100%** of double-booking is prevented and only one transaction reserves the shared resource  
**And** the unsuccessful transaction shall roll back in full, leave its task in the `Ready` state, and leave all other resources in its package `Available` (zero partial reservations)  
**And** both allocation transactions (the successful commit and the rolled-back conflict) shall complete within a maximum of **1.5 seconds** from their respective server receipt times  
**And** the unsuccessful Dispatcher shall be notified that the allocation failed and their screen shall refresh to the current state.

---

### TEST-05: Automatically fall back to the next-best available alternative resource on conflict
**Given** two Dispatchers concurrently submit automatic allocation requests (by geographic proximity) for two different `Ready` tasks requiring a bulldozer, and multiple `Available` bulldozers exist in the pool  
**When** both automatic allocation requests initially select the same closest bulldozer simultaneously  
**Then** the first transaction shall lock and reserve the initially selected bulldozer  
**And** without requiring the second Dispatcher to re-submit the request, the second allocation process shall automatically fall back to the next-best `Available` alternative bulldozer (within **2 alternative attempts**) and reserve the complete resource package atomically without leaving any resource partially reserved  
**And** the entire second allocation process—including initial conflict handling, fallback selection, and final package reservation—shall complete within a maximum of **1.5 seconds** from the server's receipt of the request, transitioning the task to `Active`  
**And** the Dispatcher shall be able to initiate the automatic allocation from the live status screen in no more than **3 mouse clicks**.
---

### TEST-06: Roll back automatic allocation when no alternative resource is available after 2 alternative attempts
**Given** an automatic allocation request is being processed for a `Ready` task requiring a bulldozer, and the initially selected bulldozer as well as the candidates in up to **2 alternative attempts** become unavailable due to concurrent allocations  
**When** the automatic allocation process exhausts the permitted **2 alternative attempts** without finding an `Available` bulldozer  
**Then** the system shall roll back the allocation transaction in full within **1.5 seconds**, leave the task in the `Ready` state, and ensure **0** resources in the package remain partially reserved by that task  
**And** the system shall notify the Dispatcher that automatic allocation failed due to lack of available alternatives.

---

### TEST-07: Successfully report task completion, release resources, and promote dependent tasks
**Given** Task A is in the `Active` state with Field Commander X as its designated commander, dependent Task B is in the `Blocked` state waiting solely on Task A, and authorized client screens are viewing the live event  
**When** Field Commander X reports Task A as completed from the live event status screen  
**Then** the system shall transition Task A to `Completed` and immediately release all operational resources assigned to Task A back to `Available` while preserving the `Broken` status of any faulty equipment  
**And** the system shall automatically transition dependent Task B from `Blocked` to `Ready`  
**And** the system shall record the task status transition in the Audit Log with the exact timestamp and Field Commander X's user ID  
**And** the system shall push the updated task and resource states to all relevant connected screens within **1.0 second**, with Field Commander X initiating the completion report in no more than **3 mouse clicks**.

---

### TEST-08: Visually highlight an overdue active task
**Given** a task is in the `Active` state and its estimated duration has been recorded  
**When** the task's elapsed execution time exceeds its estimated duration without being completed  
**Then** the system shall visually highlight the overdue task on all relevant live event status screens within **1.0 second**.

---

### TEST-09: Reject unauthorized task completion attempt
**Given** an `Active` task is assigned to Field Commander A as its designated commander  
**When** Field Commander B (who is not the designated commander for that task) attempts to report the task as completed  
**Then** the system shall reject **100%** of the unauthorized completion request  
**And** the task shall remain in the `Active` state and all resources assigned to the task shall remain reserved.

---

### TEST-10: Generate and display an on-demand SLA report for a selected past month
**Given** the System Administrator is authenticated and has selected a past month containing eligible completed events across multiple emergency event types with recorded opening timestamps and final-task completion timestamps  
**When** the System Administrator submits a request to generate the SLA (Time Adherence) report  
**Then** the system shall generate and display the SLA report within a maximum of **5.0 seconds**  
**And** the report shall display the average elapsed time from event opening to completion of the final task, grouped by emergency event type, for the selected reporting period.

---

### TEST-11: Automatically generate monthly reports at the end of the calendar month
**Given** the system clock reaches the end of a calendar month  
**When** the Monthly Report Scheduler triggers the monthly reporting process  
**Then** the system shall automatically generate the monthly Resource Utilization and SLA (Time Adherence) reports for the completed calendar month  
**And** the generated reports shall be made available for viewing by the System Administrator.

---

### TEST-12: Handle an empty SLA report period gracefully
**Given** the System Administrator is authenticated and selects a past month containing **0** eligible events for the SLA calculation  
**When** the System Administrator requests the SLA report  
**Then** the system shall display a clear empty-state message within **0.5 seconds (500 ms)** indicating that no eligible data is available for the selected period  
**And** the client application shall remain usable and allow the System Administrator to select another reporting period.

---

### TEST-13: Recover gracefully when an SLA report query exceeds the 5.0-second timeout
**Given** the System Administrator requests an on-demand SLA report for a selected past month  
**When** the report generation query exceeds the **5.0-second** timeout threshold  
**Then** the system shall display a clear timeout message within **0.5 seconds (500 ms)** after timeout detection  
**And** the client application shall remain usable and allow the System Administrator to retry the request or select another reporting period.

---

### TEST-14: Restrict administrative report access to System Administrator only
**Given** a user is authenticated with a role other than `System Administrator` (such as `Dispatcher`, `Field Commander`, or `Logistics & Resources Officer`)  
**When** the user attempts to request or view an administrative SLA or Resource Utilization report  
**Then** the system shall reject **100%** of the unauthorized request and shall not display the report.

---

### TEST-15: Push real-time task and resource status updates to subscribed screens
**Given** authorized Dispatcher and Field Commander client screens are connected and viewing an open emergency event  
**When** a successful operation changes the state of a task or the availability status of a resource in that event  
**Then** the system shall push the updated status via Server Push to all connected, authorized client screens subscribed to that event within **1.0 second** without client polling.