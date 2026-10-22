# Rule Architecture & Data Transformation Strategy

## 1. The Challenge of Raw HR Data
In a standard IAM deployment, an authoritative source rarely provides perfectly formatted data. If raw data is ingested directly into IdentityIQ without validation, it results in messy Identity Cubes, failed downstream provisioning, and broken reporting structures. 

To build a robust Identity lifecycle, we must intercept, cleanse, and logically link the HR data in-flight. This requires three distinct layers of SailPoint rules:

## 2. Inbound Layer: Customization Rule (Data Cleansing)
A Customization Rule fires on the Application connector immediately after a row is read from the CSV, but before it reaches the Identity Profile. It allows us to modify the `ResourceObject` directly. 

Our business requirements dictate three data transformations:
* **Name Formatting:** HR data often contains inconsistent casing (e.g., `jOhN`). The rule must intercept `firstName` and `lastName` and convert them to Proper Case (e.g., `John`).
* **Cost Center Fallback:** If the `costCenter` attribute is `null` or empty, the rule must inject a default value of `CC-000` to prevent downstream null-pointer exceptions.
* **Status Normalization:** HR uses free-text statuses (`A`, `Active`, `active`, `T`, `Terminated`, `Leave`). The rule must evaluate the string and normalize it to a strict binary: `Active` or `Inactive`.

## 3. Profile Layer: Identity Creation Rule (Cube Generation)
The Identity Creation Rule bridges the HR application to SailPoint's global Identity Cubes. It dictates exactly how a brand-new user is birthed into the system.

Instead of just copying the `employeeId`, this rule will:
* Generate a standard network username using the first initial and last name (e.g., `SConnor`).
* Perform **Collision Detection** by querying the SailPoint database. If a user named John Smith already owns `JSmith`, the rule will automatically append an incremental integer (`JSmith1`, `JSmith2`).

## 4. Hierarchy Layer: Manager Correlation Rule
IdentityIQ relies on the Manager Correlation Rule to dynamically build the organizational reporting hierarchy (the "Org Chart"). 

When a user is aggregated, the HR file provides their manager's Employee ID (`managerId`). This rule will take that string, query the SailPoint database for the Identity Cube that owns that exact `employeeId`, and establish the authoritative link between the employee and the manager.
