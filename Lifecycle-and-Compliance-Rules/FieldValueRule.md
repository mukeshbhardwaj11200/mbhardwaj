# FieldValue Rule: Dynamic Form & Policy Calculation

## 1. Business Objective

The FieldValue Rule executes when a Provisioning Policy or an LCM (Lifecycle Manager) Form is evaluated. Its job is to dynamically calculate and populate the default value of a field, removing the need for manual data entry and preventing human error during account creation.

## 2. Architectural Context & Use Case

When configuring target system provisioning (such as Active Directory), relying on manual data entry for critical attributes like `sAMAccountName` or `userPrincipalName` introduces a significant risk of typographical errors and naming convention violations. Manual input at the access request level degrades data integrity and often leads to downstream provisioning failures.

To enforce standardization, Field Value Rules are integrated directly into the Application Provisioning Policy. These rules programmatically extract baseline attributes (such as first and last name) from the aggregated Identity Cube, sanitize the data (e.g., stripping spaces or special characters), and dynamically generate the required corporate format. This architectural pattern ensures that mandatory account attributes are automatically and accurately populated prior to the submission of the account creation request.

## 3. Enterprise Scenarios & BeanShell Implementation

### Scenario A: Generating Standardized Active Directory IDs

**The Problem:** The Active Directory connector needs a sAMAccountName formatted as FirstInitial + LastName (e.g., JSmith), but this doesn't exist natively on the Identity cube.

**The Solution:** Intercept the identity object within the Provisioning Policy, extract the names, format them, and return the calculated string to populate the field.

```java
import sailpoint.object.Identity;

// The rule receives 'identity' (the user being provisioned) 
// and 'field' (the specific form field being evaluated).

String calculatedId = null;

if (identity != null) {
    String firstName = identity.getFirstname();
    String lastName = identity.getLastname();
    
    if (firstName != null && lastName != null) {
        
        // 1. Get first initial and full last name
        String firstInit = firstName.substring(0, 1).toLowerCase();
        String last = lastName.toLowerCase().replaceAll("\\s+", ""); // Remove spaces
        
        // 2. Combine them
        calculatedId = firstInit + last;
        
        // Note for Interviews: In a real-world scenario, you would also 
        // inject logic here to check if 'JSmith' already exists in AD, 
        // and append a number (JSmith1) if necessary.
    }
}

// 3. Return the string to populate the field in the UI/Policy
return calculatedId;
