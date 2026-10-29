# Manager Correlation Rule: Hierarchy Linking

## 1. Business Objective

The Manager Correlation Rule executes during aggregation to resolve the organizational hierarchy. Its function is to take the raw "manager" data provided by the application account (which is often messy or formatted as a Distinguished Name) and correctly link it to the actual Manager's Identity cube in SailPoint.

## 2. Architectural Context & Use Case

In complex target systems such as Active Directory, the 'manager' attribute is rarely provided as a simple Employee ID. Instead, it is typically formatted as a full Distinguished Name (DN), such as `CN=John Doe,OU=Users,DC=Corp`. SailPoint IdentityIQ cannot natively resolve this complex DN to the corresponding Manager Identity Cube without programmatic intervention.

A Manager Correlation Rule is implemented to parse these complex DN strings, extract the unique identifier (such as the Common Name or UID), and query the system to correlate the exact Manager Identity. This architectural pattern is critical for ensuring that the organizational reporting hierarchy is accurately ingested, maintained, and represented within SailPoint.

## 3. Enterprise Scenarios & BeanShell Implementation

### Scenario A: Parsing Active Directory Manager DNs

**The Problem:** Active Directory returns the manager attribute as CN=10045,OU=Managers,DC=Company,DC=com. SailPoint needs to link this to the Identity where employeeId == 10045.

**The Solution:** Use Java string manipulation to parse the Distinguished Name, extract the value inside the CN=, and return it to SailPoint for correlation.

```java
import java.util.Map;
import java.util.HashMap;

// The rule receives 'managerAttributeValue' (the raw string from the application)

Map returnMap = new HashMap();

if (managerAttributeValue != null && managerAttributeValue.contains("CN=")) {
    
    // 1. Example DN: "CN=10045,OU=Managers,DC=Company"
    // Split the string by commas
    String[] dnParts = managerAttributeValue.split(",");
    
    for (String part : dnParts) {
        
        // 2. Find the part that starts with CN=
        if (part.trim().toUpperCase().startsWith("CN=")) {
            
            // 3. Extract just the value (e.g., "10045")
            String managerId = part.substring(3).trim();
            
            // 4. Tell SailPoint to link to the Identity where employeeId matches this value
            returnMap.put("identityAttributeName", "employeeId");
            returnMap.put("identityAttributeValue", managerId);
            
            break;
        }
    }
}

return returnMap;
