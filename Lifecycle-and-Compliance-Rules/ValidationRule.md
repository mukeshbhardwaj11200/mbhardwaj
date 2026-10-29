Validation Rule: Data Integrity Check

1. Business Objective

The Validation Rule executes when a user clicks "Submit" on an LCM Form or a Provisioning Policy. Its primary function is to evaluate the input provided in a specific field, verify it meets business or security constraints, and reject the submission with an error message if it does not.

2. Architectural Context & Use Case

Maintaining strict data hygiene and security at the UI input layer is a critical architectural requirement. When users request new resources or identities—such as generic service accounts—via IdentityIQ forms, the input must be rigorously validated to prevent the submission of weak credentials or incomplete business justifications.

Validation Rules are implemented at the form-field level to programmatically enforce these governance standards. Upon form submission, the rule intercepts the input data and evaluates it against predefined logic, such as Regex patterns for password complexity. If the input fails to meet compliance standards, the rule halts the workflow and surfaces a targeted UI error message. This architectural pattern ensures that only secure, sanitized, and compliant data enters the provisioning pipeline.

3. Enterprise Scenarios & BeanShell Implementation

Scenario A: Enforcing Password Complexity on a Form

The Problem: An "Update Password" LCM form allows users to type a new password, but Active Directory will reject it later in the process if it isn't complex enough, causing workflow failures.

The Solution: Validate the value string using standard Java Regular Expressions before the form is even submitted.

import sailpoint.tools.Message;
import java.util.regex.Pattern;
import java.util.regex.Matcher;

// The rule receives 'value' (the input provided by the user)

if (value != null) {
    String password = (String) value;
    
    // 1. Define complexity: At least 8 chars, 1 uppercase, 1 number, 1 special char
    String regex = "^(?=.*[0-9])(?=.*[A-Z])(?=.*[@#$%^&+=!])(?=\\S+$).{8,}$";
    
    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(password);
    
    // 2. If it fails the regex check
    if (!matcher.matches()) {
        
        // 3. Return a standard string, or a SailPoint Message object, 
        // which renders as red error text next to the field in the UI.
        return new Message("Password must be at least 8 characters and include an uppercase letter, a number, and a special character.", null);
    }
}

// 4. Return null if validation passes (allows the form to submit)
return null;
