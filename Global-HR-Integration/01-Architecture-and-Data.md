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
E001,Sarah,Connor,sarah.connor@acme.com,Executive,Chief Executive Officer,,New York,CC-100,Active
E002,John,Smith,john.smith@acme.com,IT,IT Director,E001,New York,CC-200,Active
E003,Emily,Chen,emily.chen@acme.com,Human Resources,HR Director,E001,London,CC-300,Active
E004,David,Wallace,david.wallace@acme.com,Finance,Finance Director,E001,New York,CC-400,Active
E005,Jan,Levinson,jan.levinson@acme.com,Sales,VP of Sales,E001,Chicago,CC-500,Active
E006,Michael,Scott,michael.scott@acme.com,IT,Systems Engineer,E002,New York,CC-200,Active
E007,Jim,Halpert,jim.halpert@acme.com,IT,Helpdesk Analyst,E002,New York,CC-200,Inactive
E008,Dwight,Schrute,dwight.schrute@acme.com,IT,Security Analyst,E002,New York,CC-200,Active
E009,Angela,Martin,angela.martin@acme.com,IT,Network Administrator,E002,London,CC-200,Active
E010,Kevin,Malone,kevin.malone@acme.com,IT,Database Administrator,E002,Chicago,CC-200,Active
E011,Oscar,Martinez,oscar.martinez@acme.com,IT,Cloud Architect,E002,New York,CC-200,Active
E012,Pam,Beesly,pam.beesly@acme.com,Human Resources,HR Generalist,E003,London,CC-300,Active
E013,Toby,Flenderson,toby.flenderson@acme.com,Human Resources,HR Representative,E003,New York,CC-300,Active
E014,Kelly,Kapoor,kelly.kapoor@acme.com,Human Resources,Talent Acquisition,E003,London,CC-300,Active
E015,Holly,Flax,holly.flax@acme.com,Human Resources,HR Manager,E003,Chicago,CC-300,Active
E016,Stanley,Hudson,stanley.hudson@acme.com,Finance,Senior Accountant,E004,New York,CC-400,Active
E017,Phyllis,Vance,phyllis.vance@acme.com,Finance,Payroll Specialist,E004,New York,CC-400,Active
E018,Meredith,Palmer,meredith.palmer@acme.com,Finance,Procurement Analyst,E004,London,CC-400,Active
E019,Creed,Bratton,creed.bratton@acme.com,Finance,Quality Assurance,E004,Chicago,CC-400,Inactive
E020,Ryan,Howard,ryan.howard@acme.com,Finance,Financial Analyst,E004,New York,CC-400,Active
E021,Andy,Bernard,andy.bernard@acme.com,Sales,Sales Manager,E005,Chicago,CC-500,Active
E022,Erin,Hannon,erin.hannon@acme.com,Sales,Account Executive,E021,Chicago,CC-500,Active
E023,Gabe,Lewis,gabe.lewis@acme.com,Sales,Sales Representative,E021,New York,CC-500,Active
E024,Clark,Green,clark.green@acme.com,Sales,Junior Sales,E021,London,CC-500,Active
E025,Pete,Miller,pete.miller@acme.com,Sales,Account Executive,E021,Chicago,CC-500,Inactive
E026,Darryl,Philbin,darryl.philbin@acme.com,Sales,Logistics Coordinator,E005,New York,CC-500,Active
E027,Roy,Anderson,roy.anderson@acme.com,Sales,Warehouse Associate,E005,New York,CC-500,Active
E028,Val,Johnson,val.johnson@acme.com,Sales,Warehouse Manager,E005,London,CC-500,Active
E029,Todd,Packer,todd.packer@acme.com,Sales,Traveling Sales,E005,Chicago,CC-500,Active
E030,Jo,Bennett,jo.bennett@acme.com,Sales,Business Development,E005,New York,CC-500,Active
