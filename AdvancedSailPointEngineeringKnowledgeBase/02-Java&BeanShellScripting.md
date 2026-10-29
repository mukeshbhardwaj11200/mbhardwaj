# 2. Java & BeanShell Scripting (The Object Model)

## Architectural Context

At its core, SailPoint is a Java application. L3 engineering requires bypassing the UI and interacting directly with the SailPoint API object model (sailpoint.object.* and sailpoint.api.*) via BeanShell to manipulate identities, execute queries, and construct provisioning transactions.

## Advanced Implementation Strategies

1. Memory Management in Large Iterations: When writing custom tasks or Run Rules that iterate over hundreds of thousands of Identity or Link objects, JVM OutOfMemory (OOM) errors are a massive risk. I utilize QueryOptions, Filter, and IncrementalObjectIterator, coupled with context.decache(object) to flush the hibernate session cache and maintain a flat memory profile.

2. Programmatic Provisioning: Building ProvisioningPlan objects entirely from scratch in BeanShell. This involves instantiating AccountRequest and AttributeRequest objects, setting the Op (e.g., Add, Modify, Remove), and executing the plan programmatically via Provisioner.execute(plan).

3. Logging and Debugging: Implementing robust log4j logging within BeanShell (e.g., Log log = LogFactory.getLog("rule.custom.provisioning")) to trace variable states in production without impacting performance, utilizing isDebugEnabled() checks to prevent unnecessary string evaluation.
