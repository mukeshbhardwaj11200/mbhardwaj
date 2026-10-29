IdentityTrigger Rule: JML Automation Engine

1. Business Objective

The IdentityTrigger Rule executes during the Identity Refresh background task. Its primary function is to compare a user's previous state to their current state to detect critical changes (like a termination or a department transfer). If it returns true, it triggers a Lifecycle Event workflow (Joiner, Mover, or Leaver) to automatically provision or deprovision access.

2. Architectural Context & Use Case

A mature Identity and Access Management (IAM) architecture relies on automated, event-driven Joiner/Mover/Leaver (JML) processes rather than manual, ticket-based provisioning. The Identity Trigger Rule is the foundational component used to facilitate this automation by continuously monitoring authoritative HR feeds for critical state changes.

During the Identity Refresh task, this rule programmatically compares the `previousIdentity` object against the `newIdentity` object. By evaluating specific attribute mutations—such as a `lifecycleState` transitioning from 'Active' to 'Terminated'—the rule dynamically returns a boolean true to instantly execute the corresponding Leaver workflow. This event-driven pattern ensures that all target application accounts are immediately disabled and access is revoked, eliminating Day-1 security risks and maintaining strict access compliance.

3. Enterprise Scenarios & BeanShell Implementation

Scenario A: The Automated Leaver Event

The Problem: The company needs all IT access disabled within 2 hours of a user being marked as terminated in Workday.

The Solution: Intercept the previousIdentity and newIdentity, extract the lifecycleState attribute from both, and check if it changed to "Terminated".

import sailpoint.object.Identity;

// The rule receives 'previousIdentity' (state before refresh) 
// and 'newIdentity' (state after refresh).

boolean triggerLeaver = false;

if (previousIdentity != null && newIdentity != null) {
    
    // 1. Get the lifecycle states
    String oldState = (String) previousIdentity.getAttribute("lifecycleState");
    String newState = (String) newIdentity.getAttribute("lifecycleState");
    
    // 2. Safely evaluate the change
    if (newState != null && newState.equalsIgnoreCase("Terminated")) {
        
        // 3. Ensure we only trigger it ONCE when the change actually happens
        if (oldState == null || !oldState.equalsIgnoreCase("Terminated")) {
            triggerLeaver = true;
        }
    }
}

// 4. Return true to fire the Leaver Workflow attached to this trigger
return triggerLeaver;


Scenario B: The Cross-Department "Mover" Event

The Problem: When an employee transfers from "Sales" to "Finance", they need their Sales access revoked and Finance access granted.

The Solution: Compare the department attribute between the old and new identity cubes.

import sailpoint.object.Identity;

boolean triggerMover = false;

if (previousIdentity != null && newIdentity != null) {
    
    String oldDept = (String) previousIdentity.getAttribute("department");
    String newDept = (String) newIdentity.getAttribute("department");
    
    // Check if the department has changed (and isn't just a blank update)
    if (newDept != null && oldDept != null && !newDept.equalsIgnoreCase(oldDept)) {
        triggerMover = true;
    }
}

// Return true to fire the Mover Workflow
return triggerMover;
