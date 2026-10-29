1. Workflows & Lifecycle Management (LCM)

Architectural Context

Lifecycle Management in SailPoint is driven by the Workflow engine, which orchestrates business logic, approvals, and the compilation of ProvisioningPlan objects. Standard out-of-the-box (OOTB) workflows often fail to capture complex enterprise requirements (such as multi-tiered, risk-based approvals or conditional provisioning).

Advanced Implementation Strategies

The Provisioning Broker & Project Compilation: When an LCM request is submitted, the workflow utilizes the Provisioning Broker to compile the ProvisioningPlan into a ProvisioningProject. I heavily customize the LCM Provisioning workflow by injecting Step and Transition logic before the Provision step to dynamically alter the project based on business rules.

Dynamic Approval Routing: Implementing custom Approval steps that dynamically calculate approvers using BeanShell. For example, reading an identity's riskScore or regulatory flags (e.g., SOX) to inject a dynamic Risk Management approval step into the ApprovalSet, rather than relying on hardcoded manager/owner approvals.

Sub-Processes & Asynchronous Execution: For complex provisioning (like triggering external ticketing systems via ServiceNow), I implement asynchronous sub-processes. The workflow makes the API call and transitions to a Wait state, utilizing callbacks (WorkflowHandler) to resume execution only when the external system confirms task completion.

Forms and User Experience: Designing dynamic, multi-page UI Forms within workflows. Utilizing Field validation and AllowedValuesDefinition rules to fetch real-time data (e.g., querying an active directory schema for valid OUs) to populate dropdowns dynamically, preventing "garbage in" at the request level.
