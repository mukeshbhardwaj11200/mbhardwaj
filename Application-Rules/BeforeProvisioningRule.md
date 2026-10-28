# Before Provisioning Rule: Plan Interception & Enrichment

## 1. Business Objective

The Before Provisioning Rule executes after SailPoint has compiled the ProvisioningPlan but right before it is dispatched to the target application's connector. Its primary function is to intercept the plan to enrich it with missing data (like default passwords or calculated IDs), validate operations, or cancel unauthorized requests.

## 2. Architectural Context (The Interview Talking Track)

When asked about provisioning lifecycles, this rule is your golden ticket to demonstrating enterprise IAM experience.

"IdentityIQ's native role model is great, but it often lacks application-specific context during account creation. For example, when SailPoint creates an Active Directory account, it doesn't natively know how to generate a complex temporary password, or it might be missing a required birthright group. I use the Before Provisioning Rule to intercept 'Create' operations, dynamically generate attributes like userPrincipalName or temporary passwords, and inject them directly into the Provisioning Plan before it hits the Active Directory connector. This ensures the account creation never fails due to missing mandatory attributes."

## 3. Enterprise Scenarios & BeanShell Implementation

### Scenario A: Injecting Mandatory Active Directory Attributes

**The Problem:** An automated role provisions a user to Active Directory. However, AD requires a default password and a userPrincipalName to successfully create the account, which the standard role does not provide.

**The Solution:** Intercept the plan, isolate the AccountRequest for Active Directory, verify it is an account creation (AccountRequest.Operation.Create), and inject the missing AttributeRequests.

```java
import sailpoint.object.ProvisioningPlan;
import sailpoint.object.ProvisioningPlan.AccountRequest;
import sailpoint.object.ProvisioningPlan.AttributeRequest;
import java.util.List;

// The rule receives the 'plan' (ProvisioningPlan) and 'application' (Application)

if (plan != null) {
    // 1. Iterate through all account requests in the plan
    List<AccountRequest> accountRequests = plan.getAccountRequests();
    if (accountRequests != null) {
        
        for (AccountRequest acctReq : accountRequests) {
            
            // 2. We only want to intercept Account CREATION events
            if (AccountRequest.Operation.Create.equals(acctReq.getOperation())) {
                
                // 3. Extract the native identity (usually the sAMAccountName for AD)
                String nativeIdentity = acctReq.getNativeIdentity();
                
                if (nativeIdentity != null) {
                    // 4. Generate and inject the userPrincipalName
                    String upn = nativeIdentity + "@crisil.corp";
                    acctReq.add(new AttributeRequest("userPrincipalName", upn));
                    
                    // 5. Inject a secure, default temporary password
                    acctReq.add(new AttributeRequest("*password*", "Welcome@2026!"));
                    
                    // 6. Force the user to change the password at next logon
                    acctReq.add(new AttributeRequest("pwdLastSet", true));
                }
            }
        }
    }
}

// 7. Return the modified plan back to the provisioning engine
// Note: Some connector versions do not require you to return the plan, 
// but it is a safe best-practice.
