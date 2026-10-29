# Build Map Rule: Raw Data Transformation

## 1. Business Objective

The Build Map Rule is specific to Delimited File (CSV) and JDBC connectors. It executes during aggregation, exactly at the moment a line of data is read from the file or database, but before SailPoint creates the ResourceObject. Its function is to map the raw string array into a usable Map of key-value pairs, allowing for on-the-fly data transformation.

## 2. Architectural Context & Use Case

When integrating with legacy HR flat files or legacy databases, the raw data structure frequently lacks the normalization required by SailPoint. For instance, a source CSV might provide separate 'First Name' and 'Last Name' columns without a required 'Full Name' attribute, or it might utilize cryptic status codes (e.g., 'A' and 'T').

The Build Map Rule serves as the initial data transformation layer. It is implemented to parse the raw record, manipulate and merge attributes, translate legacy codes into standardized values, and construct a clean, normalized Map object. This architectural pattern acts as the primary line of defense for data integrity, ensuring that only sanitized, structured data enters the IdentityIQ processing pipeline.

## 3. Enterprise Scenarios & BeanShell Implementation

### Scenario A: Combining Columns and Translating Status Codes

**The Problem:** A delimited file application provides FName and LName, but you need DisplayName. Also, it provides account status as A (Active) or L (Leave), which SailPoint doesn't understand natively.

**The Solution:** Intercept the raw record (a List of Strings) and the cols (the column headers). Build the map manually, format the display name, and translate the status to the standard IIQDisabled flag.

```java
import java.util.Map;
import java.util.HashMap;

// The rule receives 'record' (List of row values), 'cols' (List of header names)

Map map = new HashMap();

if (cols != null && record != null && cols.size() == record.size()) {
    
    // 1. Initially map all raw columns to their values
    for (int i = 0; i < cols.size(); i++) {
        map.put(cols.get(i), record.get(i));
    }
    
    // 2. Synthesize a new "DisplayName" attribute
    String fName = (String) map.get("FName");
    String lName = (String) map.get("LName");
    
    if (fName != null && lName != null) {
        map.put("DisplayName", fName.trim() + " " + lName.trim());
    }
    
    // 3. Translate legacy Status codes
    String status = (String) map.get("Status");
    if (status != null) {
        if (status.equalsIgnoreCase("A")) {
            map.put("IIQDisabled", false); // Active
        } else if (status.equalsIgnoreCase("L") || status.equalsIgnoreCase("T")) {
            map.put("IIQDisabled", true);  // Leave/Terminated = Disabled
        }
    }
}

// 4. Return the finalized map to SailPoint to create the ResourceObject
return map;
