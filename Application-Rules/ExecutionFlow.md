# Application Rules Execution Flow

## 1. The Two Lifecycles

In SailPoint IdentityIQ, the application rules shown in the configuration UI do not all run at the same time. They are divided into two distinct pipelines:

The Aggregation Pipeline (Read): Pulling data from the target application and building Identity Cubes.

The Provisioning Pipeline (Write): Sending access changes from IdentityIQ out to the target application.

## 2. The Aggregation Pipeline (Data Inbound)

During an account aggregation task, data flows from the raw target system into IdentityIQ. The rules execute in a strict sequence to parse, clean, and link the data.

graph TD
    RawData[(Raw Target Data)] --> |1. Connector Reads Data| ConnectorRules

    subgraph ConnectorRules [1. Connector Rules (Parsing)]
        BM[Build Map Rule] --> MM[MergeMaps Rule]
        MM --> MRO[Map To ResourceObject Rule]
    end

    MRO --> |2. Raw Map Converted| SchemaRules

    subgraph SchemaRules [2. Schema Rules (Initial Scrub)]
        SC[Account Customization Rule]
    end

    SC --> |3. Clean ResourceObject| AggregationRules

    subgraph AggregationRules [3. Aggregation Rules (Linking & Processing)]
        AC[App Customization Rule] --> CR[Correlation Rule]
        CR -->|Identity Match Found| MCR[Manager Correlation Rule]
        CR -->|No Match Found| Create[Creation Rule]
        Create --> MCR
    end

    MCR --> IdentityCube((Identity Cube Saved))
    
    classDef rules fill:#0a4b78,stroke:#fff,stroke-width:2px,color:#fff;
    class BM,MM,MRO,SC,AC,CR,Create,MCR rules;


## Execution Order (Aggregation):

### Connector Rules: (Runs first, per row of data)

Build Map: Takes a raw string/CSV row and turns it into a Java Map.

MergeMaps: If the user has multiple rows (e.g., multiple groups), it merges them into a single Map.

Map to ResourceObject: Converts the Java Map into a SailPoint ResourceObject.

### Schema Rules:

Schema Customization Rule: Modifies the raw ResourceObject (e.g., standardizing phone numbers) before IIQ's core engine touches it.

Aggregation Rules: (Identity Engine takes over)

App Customization Rule: Acts identically to the Schema Customization rule (just applied at the app level instead of the specific schema level).

Correlation Rule: Takes the clean ResourceObject and attempts to link it to an existing Identity Cube (e.g., matching EmployeeID).

Creation Rule: If Correlation fails, and the app is an Authoritative Source, this rule runs to initialize a brand new Identity Cube.

Manager Correlation Rule: Finally, parses the manager string to link the account to the correct manager's Identity.

## 3. The Provisioning Pipeline (Data Outbound)

When a user requests access, or an automated birthright role is assigned, IdentityIQ builds a ProvisioningPlan and sends it to the target application.

graph TD
    Trigger((Access Request / Lifecycle Event)) --> |Generates| Plan[Provisioning Plan]

    Plan --> |1. Plan Compiled| ProvRulesPre

    subgraph ProvRulesPre [1. Provisioning Rules (Pre-Flight)]
        BP[Before Provisioning Rule]
    end

    BP --> |2. Enriched Plan| ConnRules

    subgraph ConnRules [2. Connector Rules (Execution)]
        PR[Provision Rule]
    end

    PR --> |3. Updates Target| Target[(Target System)]

    Target --> |4. Returns Result| ProvRulesPost

    subgraph ProvRulesPost [3. Provisioning Rules (Post-Flight)]
        AP[After Provisioning Rule]
    end
    
    classDef rules fill:#0a4b78,stroke:#fff,stroke-width:2px,color:#fff;
    class BP,PR,AP rules;


Execution Order (Provisioning):

1. Before Provisioning Rule: Executes right before the plan leaves SailPoint. Used to inject missing data (like a default password, or a calculated AD sAMAccountName) into the plan so it doesn't fail.

2. Connector Rules (Provision Rule): The actual execution script (often used for JDBC, Web Services, or custom integrations) that takes the ProvisioningPlan and translates it into SQL inserts or API calls to physically create the account.

3. After Provisioning Rule: Executes after the target system confirms the operation. Used to log success/failures, send emails, or trigger ITSM tickets (like ServiceNow) if manual intervention is still needed.
