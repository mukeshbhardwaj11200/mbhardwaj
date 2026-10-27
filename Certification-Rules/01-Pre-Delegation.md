Certification Pre-Delegation Rule: Automated Campaign Routing

1. Business Objective

The Certification Pre-Delegation Rule executes during the generation phase of an Access Review, after the campaign is built but before the notification emails are sent to the reviewers.

Its primary function is to programmatically intercept the certification entities (the users being reviewed) and automatically redirect them to a different reviewer before the campaign goes live.

2. Architectural Context (The Interview Talking Track)

If an interviewer asks you why and how you use this rule, your answer should focus on Exception Handling and Regulatory Routing.

"Standard certifications rely on a rigid HR hierarchy, but enterprise organizations frequently have exceptions. If a manager owns a team that includes external contractors, or if the manager themselves goes on unexpected medical leave, a standard campaign will either violate policy by sending the review to the wrong person, or stall out in an inactive inbox. I use the Pre-Delegation rule to intercept the campaign generation, check the employee type or the certifier's status, and dynamically route those specific items to the Vendor Management Office or an Escalation Admin."

3. Enterprise Scenarios & BeanShell Implementation

Scenario A: The Contractor / Vendor Split

The Problem: In Manager Campaigns, managers are often assigned to review external contractors (non-employees) who roll up to them in the HR hierarchy. However, corporate audit policy dictates that the Vendor Management Office (VMO) must review all contractor access, not the functional manager.

The Solution: Intercept the entity, check the employeeType Identity attribute, and if it equals "Contractor", reassign the entire review for that specific user to the VMO workgroup.

import sailpoint.object.CertificationEntity;
import sailpoint.object.Identity;
import java.util.Map;
import java.util.HashMap;

// 1. Get the Identity object of the user being certified
Identity targetIdentity = entity.getIdentity();

// 2. Safely extract the employment type
String employeeType = (String) targetIdentity.getAttribute("employeeType");

// 3. Evaluate if the user is a Contractor
if (employeeType != null && employeeType.equalsIgnoreCase("Contractor")) {
    
    // 4. Build the delegation map expected by IdentityIQ
    Map results = new HashMap();
    
    // Assign to the specific VMO Workgroup
    results.put("recipientName", "VMO_Governance_Workgroup");
    
    // Add audit context so the new reviewer knows why they got this
    results.put("description", "Automated Reassignment: Contractor access review routed to VMO.");
    results.put("comments", "Per audit policy, Contractor reviews are managed centrally by the VMO.");
    
    // Use 'reassign' (true) to transfer full ownership, rather than 'delegate' (false)
    results.put("reassign", true); 
    
    return results;
}

// 5. Return null if it is a standard employee, leaving it with the original manager
return null;


Scenario B: The "Inactive Certifier" Fallback

The Problem: A periodic campaign launches, but the manager who is supposed to review the team was terminated or put on long-term leave yesterday. If the campaign launches, it will sit in an inactive user's inbox, causing a compliance violation when the deadline is missed.

The Solution: Check the status of the certifier (the manager). If the certifier is inactive, dynamically route the team's review to a central IAM Compliance Team so it doesn't get lost.

import sailpoint.object.CertificationEntity;
import sailpoint.object.Identity;
import sailpoint.object.Certification;
import java.util.Map;
import java.util.HashMap;

// 1. Extract the primary certifier (the manager) from the campaign context
Identity currentCertifier = certification.getCertifier();

// 2. Check if the certifier is inactive (terminated, on leave, etc.)
if (currentCertifier != null && currentCertifier.isInactive()) {
    
    Map results = new HashMap();
    
    // Route to a fallback administrative workgroup
    results.put("recipientName", "IAM_Compliance_Admins");
    results.put("description", "Automated Fallback: Original manager is inactive.");
    results.put("comments", "Routed to compliance queue because " + currentCertifier.getName() + " is currently inactive.");
    
    results.put("reassign", true);
    
    return results;
}

return null;
