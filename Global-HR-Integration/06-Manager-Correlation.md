# Hierarchy Generation: Manager Correlation Rule

## 1. Business Objective
IdentityIQ relies on a dynamically generated organizational chart to route approval workflows, trigger manager-level certification campaigns, and enforce Segregation of Duties (SoD) policies. 

The HR authoritative feed provides a `managerId` for each employee. The objective of the `ManagerCorrelation` rule is to intercept this string value and instruct SailPoint on how to query its internal database to find the exact Identity Cube that matches this manager, linking the two identities in a Manager/Subordinate relationship.

## 2. Engineering Logic
Instead of querying the SailPoint database directly via `context.getObjectByName()`, a Manager Correlation rule is designed to return a `HashMap` containing the search criteria. SailPoint's internal engine then executes the query.

1. **Validation:** Check if the incoming `managerAttributeValue` is null or empty (e.g., top-level executives who do not report to anyone). If so, return `null`.
2. **Map Construction:** Instantiate a `HashMap`.
3. **Targeting:** Inject the `"identityAttributeName"` key with a value of `"employeeId"` to tell SailPoint which Identity attribute acts as the primary key.
4. **Resolution:** Inject the `"identityAttributeValue"` key with the actual HR string.

## 3. BeanShell Implementation
```java
import java.util.Map;
import java.util.HashMap;

String mgrId = managerAttributeValue;

if (mgrId != null && mgrId.length() > 0) {
    Map returnMap = new HashMap();
    
    returnMap.put("identityAttributeName", "employeeId");
    returnMap.put("identityAttributeValue", mgrId);
    
    return returnMap;
}

return null;
