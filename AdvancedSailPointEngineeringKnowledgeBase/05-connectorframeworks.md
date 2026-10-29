5. Connector Frameworks & Application Onboarding

Architectural Context

Connecting to heterogeneous legacy and cloud applications requires deep knowledge of the connector frameworks, specifically overriding default behaviors to handle non-standard authentication, pagination, and data structures.

Advanced Implementation Strategies

Web Services (REST/SOAP) Connector:

Pagination: Configuring complex offset, cursor-based, or link-header pagination logic in the connector configuration to iterate through massive JSON arrays.

Before/After Operation Rules: Intercepting HTTP requests/responses. For example, an After Operation Rule to parse a deeply nested, non-standard JSON response (jsonPath manipulation), extract the relevant array of users, and construct a flat Map that SailPoint can process.

JDBC Connector & Delta Aggregations:

Writing complex JOIN queries for account and group SQL queries.

Implementing highly efficient Delta Aggregations by utilizing the $(sys_last_updated) token to only pull records modified since the last aggregation timestamp, reducing aggregation time from hours to minutes.

Custom Connectors (OpenConnector): For proprietary mainframes or legacy systems, developing custom connector classes that implement sailpoint.connector.Connector to define exactly how the system authenticates, iterates over accounts, and executes provisioning commands.
