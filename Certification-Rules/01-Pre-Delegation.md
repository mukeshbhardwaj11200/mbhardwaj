vCertification Rule 1: Pre-Delegation

1. Rule Definition & Business Value

The Certification Pre-Delegation Rule executes during the generation phase of an Access Review, after the campaign is built but before the notification emails are sent to the reviewers.

The Business Problem: Standard certifications (like a Manager Campaign) rely on a rigid hierarchy. However, enterprise organizations frequently have exceptions. For example, a manager might own a team that includes external contractors, or the manager themselves might be on medical leave.

The Engineering Solution: This rule allows an IAM engineer to programmatically intercept the certification entities (the users being reviewed) and automatically redirect them to a different reviewer (like a Vendor Management group or an Escalation Admin) before the campaign ever goes live. This prevents campaign stagnation and ensures regulatory compliance.

2. Real-World Scenarios

Scenario A: The Contractor / Vendor Split

The Use Case: In Manager Campaigns, managers are often assigned to review external contractors (non-employees) who roll up to them in the HR hierarchy. However, corporate policy dictates that the Vendor Management Office (VMO) must review all contractor access, not the functional manager.

The Logic: We intercept the entity, check the employeeType Identity attribute, and if it equals "Contractor", we reassign the entire review for that specific user to the VMO workgroup.

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


Scenario B: The "Inactive Certifier" Fallback (Orphaned Campaign Prevention)

The Use Case: A periodic campaign launches, but the manager who is supposed to review the team was terminated or put on long-term leave yesterday. If the campaign launches, it will sit in an inactive user's inbox, causing a compliance violation when the deadline is missed.

The Logic: We check the status of the certifier (the manager). If the certifier is inactive, we dynamically route the team's review to a central IAM Compliance Team so it doesn't get lost.

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
