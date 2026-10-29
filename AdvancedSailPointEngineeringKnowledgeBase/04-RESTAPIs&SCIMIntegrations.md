# 4. REST APIs & SCIM Integrations

## Architectural Context

Enterprise IAM must support "headless" operations, integrating seamlessly with CI/CD pipelines, HR Information Systems (HRIS), and external ITSM tools via RESTful web services.

## Advanced Implementation Strategies

1. Inbound API Architecture:

1.1. Leveraging SailPoint's /v3 and /beta APIs (ISC) and /cc/api endpoints (IIQ) to trigger aggregations, submit access requests, or query identity metadata programmatically.

1.2. Implementing secure authentication utilizing Personal Access Tokens (PATs) in ISC or OAuth 2.0 / Basic Authentication in IIQ.

2. SCIM 2.0 Integration: Configuring the SCIM connector for target systems that support the protocol. Translating SailPoint account schemas into standard SCIM core schemas (urn:ietf:params:scim:schemas:core:2.0:User).

3. Pagination and Rate Limiting: Designing API scripts (Python/Java) that interact with SailPoint APIs utilizing offset and limit parameters for pagination, while implementing exponential backoff algorithms to gracefully handle HTTP 429 Too Many Requests responses.
