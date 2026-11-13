#  G2_Assignment1
**Course:** Software Engineering Methods  
**System:** CrisisSync 


### Engineering Team Details
* **Student 1:** Amir Sanallah | **ID:** 212716773 | **Braude Email:** Amir.Sanallah@e.braude.ac.il
* **Student 2:** Kamal Hamdony | **ID:** 324833516 | **Braude Email:** Kamal.Hamdony@e.braude.ac.il
* **Student 3:** [Full Name] | **ID:** [ID Number] | **Braude Email:** [username@braude.ac.il]
* **Student 4:** Bar Nahman | **ID:** 207851213 | **Braude Email:** Bar.Nahman@e.braude.ac.il

---

##  Requirements Traceability Matrix (RTM)


* **Operation 1: Resource Allocation and Task Completion**

| Req ID | Requirement Description  | Type (FR/NFR) | Mapped Use Case | Test ID |
| :--- | :--- | :---: | :--- | :---: |
| **REQ-01** | The system shall accept a task's resource package only when it contains exactly one Field Commander responsible for that task. | FR | X | TEST-01 |
| **REQ-02** | The system shall allocate a complete resource package to a "Ready" task in one atomic database transaction only when all requested resources are "Available" and all task predecessors are complete; if any resource cannot be reserved, the system shall roll back the entire allocation and leave the task "Ready". | FR | X | TEST-02, TEST-01 |
| **REQ-03** | The system shall permit a Field Commander to report the completion of a task only if they are the designated commander for that specific task. | FR | X | TEST-01 |
| **REQ-04** | The system shall process and complete a resource allocation transaction within a maximum of 1.5 seconds to minimize race conditions between multiple Dispatchers. | NFR | X | TEST-01, TEST-02 |
| **REQ-05** | The system shall push task status updates to all relevant active client screens (Dispatcher and Field Commander) within 1.0 second of a successful allocation or completion report. | NFR | X | TEST-01 |

* **Operation 2: Generating the SLA (Time Adherence) Report**

| Req ID | Requirement Description  | Type (FR/NFR) | Mapped Use Case | Test ID |
| :--- | :--- | :---: | :--- | :---: |
| **REQ-06** | The system shall allow the System Administrator to request and generate an SLA (Time Adherence) report on demand for a selected past month. | FR | X | TEST-03 |
| **REQ-07** | The system shall calculate and display, by event type, the average elapsed time from each event's opening to completion of its final task in the SLA report. | FR | X | TEST-03 |
| **REQ-08** | The system shall generate and display the complete monthly SLA report within a maximum of 5 seconds from the moment the System Administrator submits the request. | NFR | X | TEST-03 |

## BDD Acceptance Tests

### TEST-01: Successful resource allocation and task completion

**Given** a task is "Ready", all its predecessors are complete, and its requested package contains exactly one Field Commander and all requested resources are "Available"  
**When** the Dispatcher allocates the package and the designated Field Commander reports the task complete  
**Then** the system shall reserve the entire package atomically, set the task to "Active", and then set it to "Completed" on the designated commander's completion report  
**And** the system shall release the task's resources and make dependent tasks "Ready" when their remaining predecessors are complete.

### TEST-02: Concurrent allocation conflict rolls back the losing package

**Given** two Dispatchers concurrently allocate different "Ready" tasks whose packages both request the same available resource, and each package also contains other available resources  
**When** both allocation requests attempt to reserve the shared resource  
**Then** only one allocation shall succeed  
**And** the unsuccessful allocation shall roll back in full, leave its task "Ready", and leave every resource in its package unreserved by that task  
**And** the unsuccessful Dispatcher shall be notified that the allocation failed.

### TEST-03: Generate an SLA report for a past month

**Given** the System Administrator has selected a past month for which completed events have opening times, final-task completion times, and event types recorded  
**When** the Administrator requests the SLA report  
**Then** the system shall generate and display the report on demand  
**And** the report shall show, for each event type, the average elapsed time from event opening to completion of its final task for the selected reporting period.

---