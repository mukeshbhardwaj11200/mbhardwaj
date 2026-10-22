# Data Cleansing: Customization Rule

## 1. Business Objective
Authoritative HR feeds frequently contain null values, inconsistent casing, and unstructured statuses. If ingested directly, this pollutes the IdentityIQ database and causes downstream provisioning failures. 

To mitigate this, I engineered a `ResourceObjectCustomization` rule. This rule intercepts the data row immediately after it is read by the connector, cleanses the attributes in-flight, and passes the normalized data to the Identity Profile.

## 2. Transformation Requirements
1. **Name Normalization:** Convert inconsistently typed names (e.g., `jOhN`) into Proper Case.
2. **Cost Center Fallback:** Detect `null` or empty cost centers and inject a default `CC-000` value to prevent NullPointerExceptions in downstream logic.
3. **Lifecycle Status Normalization:** Standardize free-text HR status codes (e.g., `A`, `active`, `Leave`) into strict binary states (`Active` or `Inactive`).

## 3. BeanShell Implementation
Below is the BeanShell code written to satisfy these requirements. Note the use of short-circuit evaluation (`||`) and left-side string literals to ensure strict null-safety during execution.

```java

import sailpoint.object.ResourceObject;

// 1. Name Formatting
String fName = (String) object.getAttribute("firstName");
String lName = (String) object.getAttribute("lastName");

if (fName != null && fName.length() > 0) {
    fName = fName.substring(0, 1).toUpperCase() + fName.substring(1).toLowerCase();
    object.put("firstName", fName);
}

if (lName != null && lName.length() > 0) {
    lName = lName.substring(0, 1).toUpperCase() + lName.substring(1).toLowerCase();
    object.put("lastName", lName);
}

// 2. Cost Center Fallback
String cCenter = (String) object.getAttribute("costCenter");

if (cCenter == null || cCenter.length() == 0) {
    object.put("costCenter", "CC-000");
}

// 3. Status Normalization
String stat = (String) object.getAttribute("status");

if ("A".equalsIgnoreCase(stat) || "Active".equalsIgnoreCase(stat)) {
    object.put("status", "Active");
} else {
    object.put("status", "Inactive");
}

// 4. Return the modified object back to SailPoint
return object;
