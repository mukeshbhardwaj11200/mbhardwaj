# Certification Sign-Off Approver Rule: Multi-Level Workflows

## 1. Business Objective

The Certification Sign-Off Approver Rule executes at the exact moment the primary reviewer clicks the "Sign Off" button. Its function is to evaluate the decisions the reviewer just made and determine if the campaign can be officially closed, or if it must be routed to a secondary executive or administrative group for a multi-level approval chain.

## 2. Architectural Context (The Interview Talking Track)

When an interviewer asks about this rule, your answer should demonstrate a deep understanding of Compliance, Segregation of Duties (SoD), and Risk Mitigation.

"In highly regulated sectors like finance and ratings, a single line-manager's approval is necessary, but it is not sufficient for sensitive data. If a manager approves an employee's access to a SOX-critical financial database, audit compliance requires a secondary sign-off. I use the Sign-Off Approver Rule to intercept the campaign upon submission, scan the manager's decisions, and if any 'High Risk' items were approved, I dynamically route the campaign to the CISO or Risk Management team for a counter-signature before provisioning occurs."

## 3. Enterprise Scenarios & BeanShell Implementation

### Scenario A: The SOX/High-Risk Secondary Escalation

**The Problem:** Managers are allowed to review their teams, but if they explicitly approve (retain) access to a SOX-critical entitlement, the IT Security team must review and agree with that decision.
**The Solution:** Iterate through the CertificationItem objects. Check if the manager's action was "Approve". If it was, and the item contains the "SOX" keyword, return a map routing the campaign to the Security Workgroup.

```java
import sailpoint.object.Certification;
import sailpoint.object.CertificationItem;
import sailpoint.object.CertificationAction;
import java.util.Map;
import java.util.HashMap;

// The rule receives the 'certification' object representing the entire campaign
// and the 'certifier' object representing the person who just clicked Sign-Off.

boolean requiresSecurityReview = false;

// 1. Loop through all the line items the manager just reviewed
for (CertificationItem item : certification.getItems()) {
    
    CertificationAction action = item.getAction();
    
    // 2. We only care about items the manager decided to KEEP (Approve)
    if (action != null && action.isApprove()) {
        
        // Extract the display name of the entitlement
        String entitlementName = "";
        if (item.getExceptionEntitlements() != null && !item.getExceptionEntitlements().isEmpty()) {
            entitlementName = item.getExceptionEntitlements().get(0).getDisplayName();
        } else if (item.getBundle() != null) {
            entitlementName = item.getBundle(); // It's a role
        }
        
        // 3. Check for the critical compliance keyword
        if (entitlementName != null && entitlementName.toUpperCase().contains("SOX_CRITICAL")) {
            requiresSecurityReview = true;
            break; // Stop looping, we found at least one trigger
        }
    }
}

// 4. If triggered, build the routing map
if (requiresSecurityReview) {
    Map results = new HashMap();
    
    // Route to the Security/Compliance workgroup
    results.put("newCertifier", "IT_Security_Workgroup");
    
    // Optional: Specify a custom email template to notify the new reviewer
    results.put("emailTemplate", "Custom Secondary Sign-Off Notification");
    
    return results;
}

// 5. If no critical items were approved, return null to let the campaign close normally
return null;
```

## Scenario B: Risk-Score Based Secondary Approval

**The Problem:** The company wants to implement dynamic risk-based reviews. If an employee has an inherently high Identity Risk Score (e.g., above 800), any approval of their access must be double-checked by the Risk Office.
**The Solution:** Check the overall risk score of the target Identity. If it exceeds the threshold and the manager approved items, escalate it.

```java
import sailpoint.object.Certification;
import sailpoint.object.Identity;
import java.util.Map;
import java.util.HashMap;

// 1. Extract the target identity being certified
Identity targetIdentity = certification.getCertificationGroups().get(0).getIdentity();

if (targetIdentity != null) {
    // 2. Fetch the user's composite risk score
    int riskScore = targetIdentity.getScorecard().getCompositeScore();
    
    // 3. Evaluate against the enterprise high-risk threshold (e.g., 800)
    if (riskScore > 800) {
        
        // 4. Route to the Risk Office
        Map results = new HashMap();
        results.put("newCertifier", "Enterprise_Risk_Office");
        return results;
    }
}

return null;
