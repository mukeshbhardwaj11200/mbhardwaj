# MergeMaps Rule: Multi-Valued Flat File Aggregation

## 1. Business Objective

The MergeMaps Rule is a highly specialized rule used primarily in Delimited File connectors. It is required when a target system provides entitlement data using multiple rows per user (e.g., User A is listed three times because they have three different groups). Its function is to combine the maps of these multiple rows into a single ResourceObject containing a List of multi-valued attributes.

## 2. Architectural Context (The Interview Talking Track)

Interviewers love this question because it proves you have dealt with complex, poorly formatted data dumps.

"A common challenge with flat-file integrations is multi-valued attributes. If a database dump outputs one row per entitlement, SailPoint will normally overwrite the user's account data every time it reads a new row for the same user. To solve this, I configure the connector's index column, and then write a MergeMaps Rule. When the connector detects a duplicate account ID, my rule takes the new row's 'Group' attribute and appends it to a Java List in the existing mapped record, ensuring the final ResourceObject contains all entitlements accurately."

## 3. Enterprise Scenarios & BeanShell Implementation

### Scenario A: Aggregating Multiple Rows into a Single Entitlement List

**The Problem:** The CSV file has lines like:
jdoe, Active, FinanceGroup
jdoe, Active, AdminGroup
By default, SailPoint creates the account with only AdminGroup, overwriting FinanceGroup.

**The Solution:** Intercept the current map (what has already been processed) and the newObject map (the new row). Convert the Groups attribute into a List and add the new entitlement.

```java
import java.util.Map;
import java.util.List;
import java.util.ArrayList;

// The rule receives 'current' (the map from the first row read for this user)
// and 'newObject' (the map from the duplicate row just read)

// 1. Identify the attribute that holds the multi-valued data
String entitlementColumn = "Groups";

// 2. Extract the existing entitlement(s) from the 'current' map
Object existingVal = current.get(entitlementColumn);

// 3. Extract the new entitlement from the 'newObject' map
Object newVal = newObject.get(entitlementColumn);

if (newVal != null) {
    List mergedList = new ArrayList();
    
    // 4. Check if the existing value is already a List
    if (existingVal instanceof List) {
        mergedList.addAll((List) existingVal);
    } 
    // Or if it is a single String (first time merging)
    else if (existingVal instanceof String) {
        mergedList.add((String) existingVal);
    }
    
    // 5. Add the new row's entitlement (ensure no duplicates)
    if (newVal instanceof String && !mergedList.contains((String) newVal)) {
        mergedList.add((String) newVal);
    } else if (newVal instanceof List) {
        for (Object item : (List) newVal) {
            if (!mergedList.contains(item)) {
                mergedList.add(item);
            }
        }
    }
    
    // 6. Update the 'current' map with the new combined List
    current.put(entitlementColumn, mergedList);
}

// 7. Return the 'current' map which now contains the merged list
return current;
