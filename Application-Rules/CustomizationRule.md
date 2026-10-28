# Application Customization Rule: Data Scrubbing

## 1. Business Objective

The Customization Rule (sometimes called a ResourceObject Customization Rule) executes during aggregation before the application data is processed by SailPoint's correlation or identity mapping engine. Its primary job is to clean, transform, or format messy data directly on the incoming ResourceObject.

## 2. Architectural Context (The Interview Talking Track)

When discussing this rule, focus on Data Sanitization and Downstream Consistency.

"I rely on the Customization Rule to act as a data-scrubbing gateway. Often, target applications have terrible data hygiene—phone numbers are formatted in five different ways, or department names have random casing. Rather than letting that dirty data pollute the Identity cubes or break Correlation rules, I use the Customization Rule to intercept the ResourceObject, uppercase strings, strip special characters from phone numbers, and standardize dates. It ensures SailPoint only consumes clean data."

## 3. Enterprise Scenarios & BeanShell Implementation

### Scenario A: Standardizing Phone Numbers and Casing

The Problem: A legacy application sends department names as "hUmaN reSources" and phone numbers as (555) 123-4567 or 555.123.4567. This inconsistency breaks role-assignment rules that look for exact department matches.

The Solution: Intercept the object (the ResourceObject), clean the phone number to a numeric-only string, and convert the department name to Proper Case.

import sailpoint.object.ResourceObject;

// The rule receives 'object' (the ResourceObject containing the raw app data)

if (object != null) {
    
    // 1. Standardize Department Name (Uppercase)
    String dept = object.getStringAttribute("department");
    if (dept != null) {
        object.put("department", dept.trim().toUpperCase());
    }
    
    // 2. Standardize Phone Number (Strip all non-numeric characters)
    String phone = object.getStringAttribute("phoneNumber");
    if (phone != null) {
        // Regex: Replace anything that is NOT a digit with an empty string
        String cleanPhone = phone.replaceAll("[^0-9]", "");
        object.put("phoneNumber", cleanPhone);
    }
    
    // 3. Dynamically flag disabled accounts based on a raw status string
    String status = object.getStringAttribute("accountStatus");
    if (status != null && status.equalsIgnoreCase("Suspended")) {
        // IIQ natively understands IIQDisabled as the indicator for an inactive account
        object.put("IIQDisabled", true);
    }
}

// Return the modified ResourceObject back to the aggregation task
return object;
