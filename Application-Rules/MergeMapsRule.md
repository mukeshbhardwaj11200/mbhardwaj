# MergeMaps Rule: Multi-Valued Flat File Aggregation

## 1. Business Objective

The MergeMaps Rule is a highly specialized rule used primarily in Delimited File connectors. It is required when a target system provides entitlement data using multiple rows per user (e.g., User A is listed three times because they have three different groups). Its function is to combine the maps of these multiple rows into a single ResourceObject containing a List of multi-valued attributes.

## 2. Architectural Context & Use Case

A common architectural challenge during flat-file and database integrations is handling multi-valued attributes distributed across multiple rows. For example, if a database export outputs one row per entitlement, SailPoint's default behavior is to overwrite the user's account data upon reading each subsequent row for the same identity.

To resolve this, the connector's index column is configured in conjunction with a Merge Maps Rule. When the aggregation process detects a duplicate account ID, this rule intercepts the operation, extracts the new row's entitlement attribute (such as a 'Group' or 'Role'), and dynamically appends it to a Java List within the existing mapped record. This architectural pattern ensures that the final aggregated `ResourceObject` accurately retains all multi-valued entitlements without data loss.

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
