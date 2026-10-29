Advanced Policy Rule: Complex Segregation of Duties (SoD)

1. Business Objective

The Advanced Policy Rule executes during an Identity Refresh or Policy Check task. While standard SoD policies in the UI can easily check if a user has "Role A" and "Role B", the Advanced Policy Rule is required when the toxic combination involves complex data logic—such as evaluating cross-application entitlements, specific account statuses, or Identity attributes.

2. Architectural Context & Use Case

Basic Segregation of Duties (SoD) checks—such as preventing a user from simultaneously holding conflicting financial roles—are easily handled by native UI policies. However, enterprise "toxic combinations" frequently involve complex, cross-application variables. For example, strict audit compliance may dictate that an identity cannot hold a sensitive SAP Finance role *if* their Active Directory account belongs to a Vendor group *and* their HR record classifies them as a Contractor.

Standard UI-based policy builders lack the capability to evaluate this level of multi-dimensional conditional logic. An Advanced Policy Rule is implemented to programmatically scan the aggregated Identity Cube and evaluate these cross-application attributes and entitlements simultaneously. If a restricted combination is detected, the rule dynamically generates a `PolicyViolation` object. This architectural pattern ensures robust, automated enforcement of complex regulatory constraints and advanced SoD requirements.

3. Enterprise Scenarios & BeanShell Implementation

Scenario A: Cross-Application Toxic Combinations

The Problem: Regulatory compliance states that no user can hold the SAP_Payment_Approver entitlement in SAP while also holding the Domain Admins group in Active Directory.

The Solution: Intercept the identity, inspect its application links (accounts), and search for both specific entitlements. If both exist, build and return a PolicyViolation.

import sailpoint.object.Identity;
import sailpoint.object.PolicyViolation;
import sailpoint.object.Link;
import java.util.List;

// The rule receives 'identity' (the user being evaluated) 
// and 'policy' (the Policy object).

boolean hasSAPApprover = false;
boolean hasADAdmin = false;

// 1. Iterate through all the accounts (Links) the user owns
List<Link> links = identity.getLinks();
if (links != null) {
    for (Link link : links) {
        
        String appName = link.getApplicationName();
        
        // 2. Check SAP Account
        if (appName.equalsIgnoreCase("SAP ERP")) {
            Object roles = link.getAttribute("SAP_Roles");
            if (roles != null && roles.toString().contains("SAP_Payment_Approver")) {
                hasSAPApprover = true;
            }
        }
        
        // 3. Check AD Account
        if (appName.equalsIgnoreCase("Active Directory")) {
            Object groups = link.getAttribute("memberOf");
            if (groups != null && groups.toString().contains("Domain Admins")) {
                hasADAdmin = true;
            }
        }
    }
}

// 4. Evaluate the Toxic Combination
if (hasSAPApprover && hasADAdmin) {
    
    // 5. Generate the violation for the auditor/manager dashboard
    PolicyViolation violation = new PolicyViolation();
    violation.setIdentity(identity);
    violation.setPolicy(policy);
    violation.setDescription("Toxic Combination Detected: User holds SAP Payment Approver and AD Domain Admin privileges simultaneously.");
    violation.setStatus(PolicyViolation.Status.Open);
    
    return violation;
}

// 6. Return null if the user is compliant
return null;
