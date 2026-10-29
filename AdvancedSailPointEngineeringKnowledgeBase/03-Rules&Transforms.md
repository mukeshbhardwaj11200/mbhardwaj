3. Rules (IIQ) & Transforms (ISC)

Architectural Context

Rules and Transforms are the computation layers that sanitize, calculate, and transform data as it moves between external authoritative/target systems and the SailPoint Identity Cube.

Advanced Implementation Strategies

Identity Security Cloud (ISC) Transforms:

Implementing complex, nested JSON transforms to calculate attributes without backend code.

Utilizing Velocity templating within Transforms to manipulate string output dynamically based on if/else logic.

Creating Reference transforms to modularize logic (e.g., creating a central Calculate UPN transform that is referenced by multiple identity profiles).

IdentityIQ (IIQ) Aggregation Rules:

BuildMap Rule: Intercepting raw data rows before they become ResourceObjects. Used to map complex multi-valued strings from legacy databases into structured Java List objects.

Customization Rule: Acting as the data-scrubbing gateway. I intercept the ResourceObject to perform regex replacements, format dates to SimpleDateFormat("yyyy-MM-dd"), and drop invalid records to prevent correlation failures.

Correlation Rule: Writing multi-pass logic. If Filter.eq("employeeNumber", id) fails, the rule cascades to Filter.ignoreCase("email", email), returning a Map of correlation attributes to prevent orphaned accounts.
