IdentityTrigger Rule: JML Automation Engine

1. Business Objective

The IdentityTrigger Rule executes during the Identity Refresh background task. Its primary function is to compare a user's previous state to their current state to detect critical changes (like a termination or a department transfer). If it returns true, it triggers a Lifecycle Event workflow (Joiner, Mover, or Leaver) to automatically provision or deprovision access.

2. Architectural Context (The Interview Talking Track)

When an interviewer asks how you automate Joiner/Mover/Leaver (JML) processes, this rule is the cornerstone of your answer.

"A robust IAM system shouldn't rely on helpdesk tickets; it should be event-driven. I use IdentityTrigger rules to constantly monitor the authoritative HR feed. When the Identity Refresh task runs, my rule compares the previousIdentity cube with the newIdentity cube. If I detect that the lifecycleState has flipped from 'Active' to 'Terminated', the rule returns true, which instantly fires the Leaver workflow to disable all target application accounts and revoke access, ensuring zero day-1 security risks."

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
