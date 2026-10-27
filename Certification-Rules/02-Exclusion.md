# Certification Exclusion Rule: Strategic Campaign Filtering

## 1. Business Objective
The **Certification Exclusion Rule** executes at the very beginning of the campaign generation process. Its primary function is to evaluate raw identities, roles, and entitlements before the campaign is built, and permanently filter out specific items so they never appear on a reviewer's dashboard. 

## 2. Architectural Context (The Interview Talking Track)
If an interviewer asks you why and how you use this rule, your answer should focus on **Reviewer Fatigue** and **Audit Scope**. 

*"In enterprise environments, managers suffer from severe click-fatigue if we force them to review every single piece of access. As a senior developer, I use the Certification Exclusion Rule to surgically remove 'noise' from the campaign. I typically filter out two things: First, inactive identities and non-human service accounts that are governed by separate IAM lifecycle policies. Second, I filter out 'Birthright' access—like basic Active Directory 'Domain Users' groups—because every employee has it, it carries zero risk, and forcing a manager to certify it degrades the value of the review."*

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

Scenario B: The Birthright Noise Reduction Filter
The Problem: The campaign is flooding managers with low-level, zero-risk entitlements like "Basic Badge Access", "All-Company Mailing List", and "Domain Users".The Solution: Evaluate the certifiable object (the specific entitlement or role). If it matches your hardcoded list of birthright access, exclude it.

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
