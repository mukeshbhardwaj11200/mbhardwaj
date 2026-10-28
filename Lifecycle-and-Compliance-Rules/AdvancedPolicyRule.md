Advanced Policy Rule: Complex Segregation of Duties (SoD)

1. Business Objective

The Advanced Policy Rule executes during an Identity Refresh or Policy Check task. While standard SoD policies in the UI can easily check if a user has "Role A" and "Role B", the Advanced Policy Rule is required when the toxic combination involves complex data logic—such as evaluating cross-application entitlements, specific account statuses, or Identity attributes.

2. Architectural Context (The Interview Talking Track)

When discussing Compliance and Audit, this is your most powerful talking point.

"Basic Segregation of Duties (SoD) is easy—don't let someone be an AP Clerk and an AR Clerk. But enterprise toxic combinations are rarely that simple. A real audit finding might be: 'A user cannot have the SAP Finance role IF their Active Directory account belongs to the Vendor group AND they are marked as a Contractor in HR.' The standard UI cannot build that logic. I write Advanced Policy Rules to programmatically scan the Identity cube, evaluate all those cross-application variables at once, and generate a PolicyViolation object if the toxic combination is found."

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
