#  G2_Assignment1
**Course:** Software Engineering Methods  
**System:** CrisisSync 


### Engineering Team Details
- **Student 1:** Amir Sanallah | **ID:** 212716773 | **Braude Email:** Amir.Sanallah@e.braude.ac.il
- **Student 2:** Kamal Hamdony | **ID:** 324833516 | **Braude Email:** Kamal.Hamdony@e.braude.ac.il
- **Student 3:** [Your Full Name] | **ID:** [Your ID] | **Braude Email:** [Your Email]
- **Student 4:** Bar Nahman | **ID:** 207851213 | **Braude Email:** Bar.Nahman@e.braude.ac.il
---

##  Requirements Traceability Matrix (RTM)


- **Operation 1: Resource Allocation and Task Completion**

| Req ID | Requirement Description  | Type (FR/NFR) | Mapped Use Case | Test ID |
| :--- | :--- | :---: | :--- | :---: |
| **REQ-01** | The system shall accept a task's resource package only when it contains exactly one Field Commander responsible for that task. | FR | Manually Allocate Resources; Automatically Allocate Resources | TEST-01 |
| **REQ-02** | The system shall allocate a complete resource package to a "Ready" task in one atomic database transaction only when all requested resources are "Available" and all task predecessors are complete; if any resource cannot be reserved, the system shall roll back the entire allocation and leave the task "Ready". | FR | Manually Allocate Resources; Automatically Allocate Resources | TEST-01, TEST-02 |
| **REQ-03** | The system shall permit a Field Commander to report the completion of a task only if they are the designated commander for that specific task. | FR | Report Task Completion | TEST-01, TEST-05 |
| **REQ-04** | The system shall process and complete a resource allocation transaction within a maximum of 1.5 seconds to minimize race conditions between multiple Dispatchers. | NFR | Manually Allocate Resources; Automatically Allocate Resources | TEST-01, TEST-02 |
| **REQ-05** | The system shall push task status updates to all relevant active client screens (Dispatcher and Field Commander) within 1.0 second of a successful allocation or completion report. | NFR | Monitor Events in Real Time | TEST-01 |

- **Operation 2: Generating the SLA (Time Adherence) Report**

| Req ID | Requirement Description  | Type (FR/NFR) | Mapped Use Case | Test ID |
| :--- | :--- | :---: | :--- | :---: |
| **REQ-06** | The system shall allow the System Administrator to request and generate an SLA (Time Adherence) report on demand for a selected past month. | FR | Generate Reports | TEST-03, TEST-04 |
| **REQ-07** | The system shall calculate and display, by event type, the average elapsed time from each event's opening to completion of its final task in the SLA report. | FR | Generate Reports; View Reports | TEST-03 |
| **REQ-08** | The system shall generate and display the complete monthly SLA report within a maximum of 5 seconds from the moment the System Administrator submits the request. | NFR | Generate Reports; View Reports | TEST-03, TEST-04 |
| **REQ-09** | The system shall prevent double-booking in 100% of concurrent allocation requests by using pessimistic SQL row locking, and shall automatically retry transient database-lock failures up to 2 times before returning an explicit failure. | NFR | Manually Allocate Resources; Automatically Allocate Resources | TEST-02 |
| **REQ-10** | The system shall block 100% of task-completion attempts by a Field Commander who is not designated for that task and shall record each unauthorized attempt in the Audit Log within 200 ms. | NFR | Report Task Completion | TEST-05 |
| **REQ-11** | The system shall allow a Dispatcher to allocate resources or report task completion from the live dashboard in no more than 3 mouse clicks, and shall highlight overdue tasks within 1.0 second of their becoming overdue. | NFR | Manually Allocate Resources; Report Task Completion; Monitor Events in Real Time | TEST-01 |
                    
## BDD Acceptance Tests

### TEST-01: Successful resource allocation and task completion

**Given** a task is "Ready", all its predecessors are complete, its requested package contains exactly one Field Commander and all requested resources are "Available", and the Dispatcher is viewing the live dashboard

**When** the Dispatcher allocates the package and the designated Field Commander reports the task complete

**Then** the system shall reserve the entire package atomically, set the task to "Active", and then set it to "Completed" on the designated commander's completion report

**And** the system shall complete the allocation transaction within 1.5 seconds and push task status updates to all relevant active Dispatcher and Field Commander screens within 1.0 second of each successful allocation or completion report

**And** the system shall release the task's resources and make dependent tasks "Ready" when their remaining predecessors are complete

**And** the Dispatcher shall be able to allocate resources and report completion from the live dashboard in no more than 3 mouse clicks

**And** the system shall highlight an overdue task within 1.0 second of its becoming overdue.

### TEST-02: Concurrent allocation conflict rolls back the losing package

**Given** two Dispatchers concurrently allocate different "Ready" tasks whose packages both request the same available resource, and each package also contains other available resources

**When** both allocation requests attempt to reserve the shared resource

**Then** only one allocation shall succeed

**And** the unsuccessful allocation shall roll back in full, leave its task "Ready", and leave every resource in its package unreserved by that task

**And** the unsuccessful Dispatcher shall be notified that the allocation failed

**And** the system shall prevent double-booking for every concurrent allocation request by using pessimistic SQL row locking

**And** if an allocation encounters a transient database-lock failure, the system shall retry automatically no more than 2 times before returning an explicit failure.

### TEST-03: Generate an SLA report for a past month

**Given** the System Administrator has selected a past month for which completed events have opening times, final-task completion times, and event types recorded

**When** the Administrator requests the SLA report

**Then** the system shall generate and display the report on demand within 5 seconds

**And** the report shall show, for each event type, the average elapsed time from event opening to completion of its final task for the selected reporting period.

### TEST-04: Recover from an empty or timed-out SLA report query

**Given** the System Administrator has selected a past month with zero completed events, or the SLA report query exceeds 5 seconds

**When** the Administrator requests the SLA report

**Then** the system shall display a clear empty-state message for a month with zero completed events, or a timeout message if the query exceeds 5 seconds

**And** the system shall keep the application responsive and allow the Administrator to retry the report request.

### TEST-05: Block unauthorized task completion reporting

**Given** a task is assigned to a different Field Commander

**When** a Field Commander who is not designated for that task attempts to report it complete

**Then** the system shall block the completion attempt and leave the task's status unchanged

**And** the system shall record the unauthorized attempt in the Audit Log within 200 ms.

---