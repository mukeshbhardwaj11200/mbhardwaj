# Case Study: Global HR System Integration

## 1. Business Problem & Architecture
In enterprise Identity and Access Management (IAM), the authoritative source is the foundation of the identity lifecycle. For this integration, the business requires a new authoritative HR feed to automate the onboarding, movement, and offboarding of employees across global regions. 

The data provided by the HR system is a flat, comma-separated (CSV) file. However, as is common in real-world IAM deployments, the data contains inconsistencies: missing cost centers, varied capitalization in names, and unstructured status codes. 

Our architectural goal is to ingest this feed into SailPoint IdentityIQ, cleanse the data in-flight using a Customization Rule, automatically generate collision-proof network IDs using an Identity Creation Rule, and dynamically build the organizational reporting structure via a Manager Correlation Rule.

## 2. Application Schema
The Authoritative Application uses a `DelimitedFileConnector` configured to ingest the following standard HR schema:

| Column Name | Data Type | Description / IAM Purpose |
| :--- | :--- | :--- |
| **employeeId** | String | Unique identifier; mapped as the Identity Attribute. |
| **firstName** | String | Used for display name and username generation. |
| **lastName** | String | Used for display name and username generation. |
| **email** | String | Primary communication attribute. |
| **department** | String | Used for role-based access control (RBAC). |
| **jobTitle** | String | Used for role-based access control (RBAC). |
| **managerId** | String | Used by the Manager Correlation Rule to build reporting hierarchy. |
| **location** | String | Geographic indicator for localized access. |
| **costCenter** | String | Financial tracking (requires normalization for null values). |
| **status** | String | Lifecycle trigger (requires normalization to "Active"/"Inactive"). |

## 3. Raw HR Dataset
Below is the sample HR dataset used to validate the SailPoint integration. It contains a mix of active and terminated users, multiple departments, a multi-level reporting hierarchy, and intentional data inconsistencies that will be handled by our inbound rules.

```csv
employeeId,firstName,lastName,email,department,jobTitle,managerId,location,costCenter,status
E001,sArAh,CONNOR,sarah.connor@acme.com,Executive,Chief Executive Officer,,New York,,A
E002,John,Smith,john.smith@acme.com,IT,IT Director,E001,New York,CC-200,active
E003,Jane,Smith,jane.smith@acme.com,IT,Security Analyst,E002,New York,,Leave
E004,Jacob,Smith,jacob.smith@acme.com,IT,Helpdesk Analyst,E002,London,CC-200,A
E005,David,Wallace,david.wallace@acme.com,Finance,Finance Director,E001,New York,CC-400,Active
E006,Dan,Wallace,dan.wallace@acme.com,Finance,Senior Accountant,E005,Chicago,,T
E007,mIchaEl,scOtt,michael.scott@acme.com,Sales,VP of Sales,E001,New York,,ACTIVE
E008,Jim,Halpert,jim.halpert@acme.com,Sales,Sales Representative,E007,New York,CC-500,Active
E009,Dwight,Schrute,dwight.schrute@acme.com,Sales,Sales Representative,E007,New York,CC-500,Active
E010,Angela,Martin,angela.martin@acme.com,Finance,Payroll Specialist,E005,New York,CC-400,A
E011,Kevin,Malone,kevin.malone@acme.com,Finance,Accountant,E005,Chicago,CC-400,Leave
E012,Oscar,Martinez,oscar.martinez@acme.com,Finance,Cloud Architect,E005,New York,CC-400,active
E013,Pam,Beesly,pam.beesly@acme.com,Human Resources,HR Generalist,E001,New York,CC-300,A
E014,Toby,Flenderson,toby.flenderson@acme.com,Human Resources,HR Representative,E001,New York,CC-300,Active
E015,Kelly,Kapoor,kelly.kapoor@acme.com,Human Resources,Talent Acquisition,E001,London,CC-300,active
E016,Stanley,Hudson,stanley.hudson@acme.com,Sales,Senior Sales,E007,New York,CC-500,Terminated
E017,Phyllis,Vance,phyllis.vance@acme.com,Sales,Sales Representative,E007,New York,,active
E018,Meredith,Palmer,meredith.palmer@acme.com,Supplier Relations,Procurement,E001,London,CC-700,A
E019,Creed,Bratton,creed.bratton@acme.com,Quality Assurance,QA Analyst,E001,Chicago,CC-700,Unknown
E020,Ryan,Howard,ryan.howard@acme.com,Executive,VP of Business,E001,New York,CC-100,active
E021,Andy,Bernard,andy.bernard@acme.com,Sales,Sales Manager,E007,Chicago,CC-500,A
E022,Erin,Hannon,erin.hannon@acme.com,Human Resources,Receptionist,E001,Chicago,CC-300,Leave
E023,Gabe,Lewis,gabe.lewis@acme.com,IT,IT Coordinator,E002,Chicago,,A
E024,Clark,Green,clark.green@acme.com,Customer Service,CS Representative,E015,London,CC-600,active
E025,Pete,Miller,pete.miller@acme.com,Customer Service,CS Representative,E015,Chicago,CC-600,active
E026,Darryl,Philbin,darryl.philbin@acme.com,Warehouse,Logistics Foreman,E001,New York,,A
E027,Roy,Anderson,roy.anderson@acme.com,Warehouse,Warehouse Associate,E026,New York,CC-800,Terminated
E028,Val,Johnson,val.johnson@acme.com,Warehouse,Warehouse Manager,E026,London,CC-800,active
E029,Todd,Packer,todd.packer@acme.com,Sales,Traveling Sales,E007,Chicago,CC-500,T
E030,Jo,Bennett,jo.bennett@acme.com,Executive,Chairman,,New York,,A
