# Architectural Context

An L3 Engineer must understand the underlying database schema (hibernate mapping) to troubleshoot Severity 1 system outages, optimize performance, and write custom compliance reports.

## Advanced Implementation Strategies

**1. Schema Mastery:** Deep understanding of core tables: spt_identity (users), spt_link (accounts), spt_application (targets), spt_managed_attribute (entitlements), and spt_audit_event (logs).

**2. Performance Tuning:** Diagnosing slow identity refreshes or aggregations by running SQL execution plans. Working with DBAs to add custom indexes on highly queried extended attributes (searchable=true) in the spt_identity table.

**3. Deadlock & Transaction Management:** Identifying database deadlocks during highly concurrent provisioning operations or mass aggregations. Troubleshooting hibernate lazy-initialization exceptions and ensuring database connection pools (e.g., Tomcat context.xml MaxActive connections) are tuned accurately for 
