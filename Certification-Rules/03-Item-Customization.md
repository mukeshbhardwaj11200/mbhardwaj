Certification Item Customization Rule: UI Enrichment

1. Business Objective

The Certification Item Customization Rule executes during the final stages of campaign generation, right before the items are rendered on the reviewer's dashboard.

Its primary function is to alter the visual display of the certification line items. It does not change the underlying data in the database; it only changes what the reviewer sees on their screen.

2. Architectural Context (The Interview Talking Track)

When an interviewer asks about this rule, your focus should be on User Experience (UX) and Preventing Rubber-Stamping.

"The biggest threat to a compliance campaign is 'rubber-stamping'—when a manager blindly approves access because they don't understand what they are looking at. An Active Directory group named CN=GL_PRD_RW,OU=Groups,DC=Corp means nothing to a VP of Finance. As an IAM engineer, I use the Item Customization Rule to intercept that cryptic string and inject a plain-English 'Business Justification' (e.g., 'General Ledger Production - Read/Write') directly into the UI. I also use it to inject HTML/CSS warnings to highlight high-risk administrative access in red, forcing the manager to pay attention."

3. Enterprise Scenarios & BeanShell Implementation

Scenario A: Translating IT Jargon to Business Language

The Problem: Managers are approving cryptic Active Directory and SAP entitlement strings because they assume IT knows what they are doing.
The Solution: Intercept the item, check the application and value, and populate the Custom1 or Explanation UI columns with a readable business description.

import sailpoint.object.CertificationItem;
import sailpoint.object.CertificationEntity;

// The rule receives 'item' (the specific line item in the UI) 
// and 'entity' (the user being reviewed).

// 1. Identify the application
String appName = item.getApplication();

// 2. Process only targeted applications (e.g., Active Directory)
if (appName != null && appName.equals("Active Directory")) {
    
    // 3. Extract the entitlement value safely
    if (item.getExceptionEntitlements() != null && !item.getExceptionEntitlements().isEmpty()) {
        String entValue = item.getExceptionEntitlements().get(0).getValue();
        
        // 4. Translate cryptic names to business friendly names
        if (entValue != null) {
            if (entValue.contains("GL_PRD_RW")) {
                item.setCustom1("Finance General Ledger (Read/Write)");
                item.setExplanation("Grants ability to modify financial ledgers in Production.");
            } 
            else if (entValue.contains("HR_APP_RO")) {
                item.setCustom1("HR Application (Read-Only)");
                item.setExplanation("Grants view-only access to employee records.");
            }
        }
    }
}
// Note: This rule does not return anything. It directly modifies the 'item' object in memory.


Scenario B: Flagging High-Risk Privileged Access

The Problem: Standard access and privileged "Root/Admin" access look exactly the same in the standard SailPoint UI list, making it easy for a manager to accidentally approve dangerous permissions.
The Solution: Scan the entitlement strings for keywords like "Admin", "Root", or "Superuser". If found, inject HTML styling to make the item stand out as a high-risk decision.

import sailpoint.object.CertificationItem;

// 1. Extract the display value of the item being certified
String displayValue = item.getExceptionEntitlements() != null && !item.getExceptionEntitlements().isEmpty() 
                      ? item.getExceptionEntitlements().get(0).getDisplayName() 
                      : null;

// 2. Alternatively, check if it's a Role (Bundle)
if (displayValue == null && item.getBundle() != null) {
    displayValue = item.getBundle();
}

if (displayValue != null) {
    // 3. Convert to uppercase for easy keyword matching
    String upperValue = displayValue.toUpperCase();
    
    // 4. Look for privileged access indicators
    if (upperValue.contains("ADMIN") || upperValue.contains("ROOT") || upperValue.contains("SUPERUSER")) {
        
        // 5. Inject a highly visible HTML warning directly into the UI Explanation column
        String warningHtml = "<b style='color:red; font-size:14px;'>&#9888; HIGH RISK PRIVILEGED ACCESS</b>";
        
        // Append it if an explanation already exists, otherwise set it
        String currentExp = item.getExplanation();
        if (currentExp != null && currentExp.length() > 0) {
            item.setExplanation(warningHtml + "<br/>" + currentExp);
        } else {
            item.setExplanation(warningHtml);
        }
    }
}
