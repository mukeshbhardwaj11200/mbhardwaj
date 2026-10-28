# Application Creation Rule: Identity Initialization

## 1. Business Objective

The Creation Rule executes only during the aggregation of an Authoritative Source (like Workday, SAP HR, or Oracle HCM). When SailPoint discovers a brand new employee in the HR feed, it creates a new Identity Cube. This rule intercepts that creation event to dynamically calculate, format, or initialize foundational Identity attributes before the cube is saved to the database.

## 2. Architectural Context (The Interview Talking Track)

When asked about the Creation Rule, your answer must revolve around Authoritative Sources and Calculated Attributes.

"I use the Creation Rule exclusively on HR authoritative sources. When a new hire comes through Workday, HR usually doesn't provide a corporate email address or a standard lifecycle state—IT is expected to generate those. I use this rule to automatically calculate the user's corporate email (e.g., firstname.lastname@company.com), handle duplicate name collisions, and force the new Identity's lifecycle state to 'Pre-Hire' so downstream provisioning is triggered correctly."

## 3. Enterprise Scenarios & BeanShell Implementation

### Scenario A: Calculating Corporate Email & Lifecycle State

The Problem: The HR flat file provides First Name and Last Name, but downstream applications (like Active Directory) require a standardized Corporate Email and an active Lifecycle State to begin birthright provisioning.

The Solution: Intercept the new identity object, format the first and last names into an email string, check for uniqueness, and set the default Lifecycle State.

import sailpoint.object.Identity;
import sailpoint.object.QueryOptions;
import sailpoint.object.Filter;

// The rule receives 'identity' (the newly created, unsaved Identity cube)
// and 'account' (the raw HR ResourceObject).

// 1. Extract raw names from the HR account
String firstName = account.getStringAttribute("FirstName");
String lastName = account.getStringAttribute("LastName");

if (firstName != null && lastName != null) {
    
    // 2. Clean and format the strings (lowercase, remove spaces)
    String cleanFirst = firstName.trim().toLowerCase().replaceAll("\\s+", "");
    String cleanLast = lastName.trim().toLowerCase().replaceAll("\\s+", "");
    
    // 3. Generate base email
    String generatedEmail = cleanFirst + "." + cleanLast + "@crisil.com";
    
    // 4. (Optional in interviews) Check for uniqueness in DB here...
    
    // 5. Set the calculated attribute on the Identity cube
    identity.setAttribute("email", generatedEmail);
}

// 6. Force the initial Lifecycle State to trigger birthright provisioning rules
identity.setAttribute("lifecycleState", "PreHire");

// No return value required. The rule modifies the 'identity' object directly in memory.
