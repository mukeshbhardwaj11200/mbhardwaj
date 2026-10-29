# Application Correlation Rule: Complex Identity Linking

## 1. Business Objective

The Correlation Rule executes during account aggregation. Its primary function is to link a newly discovered application account to an existing Identity cube when the standard UI-based attribute mapping (e.g., employeeNumber == employeeId) is insufficient due to messy, missing, or complex data.

## 2. Architectural Context & Use Case

While standard UI correlation mapping suffices for systems with clean, 1:1 matching attributes (such as an exact Employee ID), legacy systems or external directories frequently contain non-standard data. For example, accounts may have hardcoded prefixes on their IDs or only provide an email address for correlation.

In these scenarios, a custom Correlation Rule is implemented to execute multi-step fallback logic. The rule programmatically sanitizes data (e.g., stripping prefixes to attempt an ID match) and cascades to secondary attributes (e.g., querying by Email) if the primary match fails. This architectural pattern ensures accurate identity aggregation and prevents the creation of duplicate, orphaned Identity Cubes within the system.

## 3. Enterprise Scenarios & BeanShell Implementation

### Scenario A: String Manipulation & Prefix Stripping

**The Problem:** A legacy financial application stores usernames with a hardcoded prefix (e.g., FIN-10045), but the authoritative Identity cube only stores the numerical ID (10045). The standard UI cannot correlate these.

**The Solution:** Intercept the account's username attribute, strip the FIN- prefix using standard Java string manipulation, and return a map containing the clean identity attribute to correlate against.

```java
import java.util.Map;
import java.util.HashMap;

// The rule receives 'account' (the ResourceObject representing the raw application data)
// and 'application' (the Application object being aggregated).

Map returnMap = new HashMap();
String rawUsername = account.getStringAttribute("username");

if (rawUsername != null && rawUsername.startsWith("FIN-")) {
    
    // 1. Strip the first 4 characters ("FIN-")
    String cleanId = rawUsername.substring(4);
    
    // 2. Tell SailPoint to look for an Identity where 'employeeId' equals this clean ID
    returnMap.put("identityAttributeName", "employeeId");
    returnMap.put("identityAttributeValue", cleanId);
}

// If returning a map, SailPoint attempts to correlate using those key-value pairs
return returnMap;
```

### Scenario B: Multi-Attribute Fallback Logic

**The Problem:** An external contractor Active Directory domain does not store Employee IDs. Some accounts have an email address, while others only have a givenName and sn (First and Last name).

**The Solution:** Extract the available attributes from the account object. Attempt to match by Email first. If the email attribute is missing, write a custom query to find the Identity by First and Last Name.

```java
import sailpoint.object.Identity;
import sailpoint.object.QueryOptions;
import sailpoint.object.Filter;
import java.util.List;
import java.util.Map;
import java.util.HashMap;

Map returnMap = new HashMap();

// 1. Extract raw attributes from the incoming account
String email = account.getStringAttribute("mail");
String firstName = account.getStringAttribute("givenName");
String lastName = account.getStringAttribute("sn");

// 2. First Fallback: Try correlating by Email
if (email != null && !email.isEmpty()) {
    returnMap.put("identityAttributeName", "email");
    returnMap.put("identityAttributeValue", email);
    return returnMap;
} 
// 3. Second Fallback: Correlate by First and Last name
else if (firstName != null && lastName != null) {
    
    // Build a filter to find an identity with both matching names
    QueryOptions qo = new QueryOptions();
    qo.addFilter(Filter.and(Filter.eq("firstname", firstName), Filter.eq("lastname", lastName)));
    
    // Search the database
    List identities = context.getObjects(Identity.class, qo);
    
    // If exactly ONE identity is found, return it directly for correlation
    if (identities != null && identities.size() == 1) {
        return identities.get(0);
    }
}

// Return null if no matches found (creates an uncorrelated account)
return null;
