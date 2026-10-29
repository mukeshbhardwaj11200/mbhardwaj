# 3. Rules (IIQ) & Transforms (ISC)

## Architectural Context

Rules and Transforms are the computation layers that sanitize, calculate, and transform data as it moves between external authoritative/target systems and the SailPoint Identity Cube.

## Advanced Implementation Strategies

### 1. Identity Security Cloud (ISC) Transforms:

#### 1.1. Implementing complex, nested JSON transforms to calculate attributes without backend code.

#### 1.2. Utilizing Velocity templating within Transforms to manipulate string output dynamically based on if/else logic.

#### 1.3. Creating Reference transforms to modularize logic (e.g., creating a central Calculate UPN transform that is referenced by multiple identity profiles).

### 2. IdentityIQ (IIQ) Aggregation Rules:

#### 2.1. BuildMap Rule: Intercepting raw data rows before they become ResourceObjects. Used to map complex multi-valued strings from legacy databases into structured Java List objects.

#### 2.2. Customization Rule: Acting as the data-scrubbing gateway. I intercept the ResourceObject to perform regex replacements, format dates to SimpleDateFormat("yyyy-MM-dd"), and drop invalid records to prevent correlation failures.

#### 2.3. Correlation Rule: Writing multi-pass logic. If Filter.eq("employeeNumber", id) fails, the rule cascades to Filter.ignoreCase("email", email), returning a Map of correlation attributes to prevent orphaned accounts.
