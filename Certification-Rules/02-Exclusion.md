# Certification Exclusion Rule: Strategic Campaign Filtering

## 1. Business Objective
The **Certification Exclusion Rule** executes at the very beginning of the campaign generation process. Its primary function is to evaluate raw identities, roles, and entitlements before the campaign is built, and permanently filter out specific items so they never appear on a reviewer's dashboard. 

## 2. Architectural Context & Use Case

In enterprise access governance, certification campaigns frequently suffer from "reviewer fatigue" when managers are forced to attest to every granular entitlement. Including ubiquitous, low-risk, or irrelevant access in a campaign degrades the overall efficacy of the audit and obscures meaningful security risks.

The Certification Exclusion Rule is implemented to programmatically refine the audit scope and surgically remove this "noise" prior to campaign generation. This architectural pattern is typically used to filter out two specific categories: first, inactive identities and non-human service accounts (which are governed by separate automated IAM lifecycle policies); and second, baseline "Birthright" access (such as the standard Active Directory 'Domain Users' group). By automatically excluding zero-risk and out-of-scope items, this rule ensures that certifiers are only presented with high-value, risk-bearing access reviews.

## 3. Enterprise Scenarios & BeanShell Implementation

### Scenario A: The Non-Human & Terminated User Filter
**The Problem:** An HR campaign is designed for managers to review their human direct reports. However, the database includes generic service accounts, RPA bots, and employees who were terminated last week (whose access is already disabled but their cubes haven't been archived yet). 
**The Solution:** Intercept the `entity` (the Identity cube), check its type and status, and return `true` to exclude it entirely.

```java
import sailpoint.object.Identity;
import sailpoint.object.AbstractCertifiableEntity;

// The rule receives 'entity' (the object being certified, usually an Identity)
// and 'certifiable' (the specific role/entitlement being evaluated).

boolean exclude = false;

// 1. Ensure we are evaluating an Identity object
if (entity instanceof Identity) {
    Identity ident = (Identity) entity;
    
    // 2. Exclude Inactive/Terminated Users
    if (ident.isInactive()) {
        exclude = true;
    }
    
    // 3. Exclude Non-Human/Service Accounts
    // Assuming 'type' is extended attribute or using the standard standard type field
    String idType = ident.getType();
    if (idType != null && (idType.equalsIgnoreCase("ServiceAccount") || idType.equalsIgnoreCase("Bot"))) {
        exclude = true;
    }
}

// Return true to drop the item from the campaign, false to include it.
return exclude;
```

### Scenario B: The Birthright Noise Reduction Filter
**The Problem:** The campaign is flooding managers with low-level, zero-risk entitlements like "Basic Badge Access", "All-Company Mailing List", and "Domain Users".The **The Solution:** Evaluate the certifiable object (the specific entitlement or role). If it matches your hardcoded list of birthright access, exclude it.

```java
import sailpoint.object.Identity;
import sailpoint.object.Bundle;
import sailpoint.object.ManagedAttribute;
import sailpoint.object.Certifiable;

boolean exclude = false;

// We only want to filter out specific items, not the whole user
if (certifiable != null) {
    
    // Check if the item being certified is a Role (Bundle)
    if (certifiable instanceof Bundle) {
        Bundle role = (Bundle) certifiable;
        String roleName = role.getName();
        
        // Exclude base employee birthright roles
        if (roleName != null && roleName.equals("Base Employee Birthright Role")) {
            exclude = true;
        }
    }
    
    // Check if the item being certified is a direct Entitlement (ManagedAttribute)
    else if (certifiable instanceof ManagedAttribute) {
        ManagedAttribute entitlement = (ManagedAttribute) certifiable;
        String entValue = entitlement.getValue();
        
        // Exclude universally safe Active Directory groups
        if (entValue != null && (entValue.equalsIgnoreCase("Domain Users") || entValue.equalsIgnoreCase("All_Company_Email"))) {
            exclude = true;
        }
    }
}

return exclude;
