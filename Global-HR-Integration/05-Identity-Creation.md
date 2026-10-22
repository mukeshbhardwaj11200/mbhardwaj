# Identity Profiling: Identity Creation Rule

## 1. Business Objective
Once the inbound HR data is cleansed, IdentityIQ must instantiate a new Identity Cube. By default, SailPoint maps a single attribute (like Employee ID) as the username. However, enterprise environments typically require a human-readable network ID (e.g., `JSmith`) for active directories and downstream systems.

The objective of this `IdentityCreation` rule is to dynamically generate a standardized username from the cleansed HR data, query the IdentityIQ database to detect potential duplicates, and automatically resolve collisions before birthing the cube.

## 2. Engineering Logic
1. **Extraction:** Pull the previously cleansed `firstName` and `lastName` from the incoming `ResourceObject`.
2. **Standardization:** Construct a base username using the first initial and the full last name, converted to lowercase.
3. **Collision Detection:** Utilize `context.getObjectByName()` to query the database. If the username exists, enter a `while` loop that appends an incremental integer to the base string until a completely unique value is found.
4. **Instantiation:** Apply the unique string to the `setName()` method of the empty Identity object.

## 3. BeanShell Implementation
```java
import sailpoint.object.ResourceObject;
import sailpoint.object.Identity;

String fName = (String) account.getAttribute("firstName");
String lName = (String) account.getAttribute("lastName");

if (fName != null && lName != null) {
    String baseName = (fName.substring(0, 1) + lName).toLowerCase();
    String uniqueName = baseName;
    int i = 1;

    while (context.getObjectByName(Identity.class, uniqueName) != null) {
        uniqueName = baseName + i;
        i++;
    }

    identity.setName(uniqueName);
}
