6. XML/JSON Object Management & CI/CD

Architectural Context

Infrastructure as Code (IaC) principles apply to SailPoint. All objects (Workflows, Rules, Applications) are serialized as XML (IIQ) or JSON (ISC). Managing these objects in version control (Git) is critical for enterprise deployment pipelines.

Advanced Implementation Strategies

Services Standard Build (SSB) & Ant: Utilizing the SSB framework in IIQ to manage deployments. Utilizing tokenization (%%ENVIRONMENT_VARIABLE%%) within XML files to ensure that credentials, URLs, and server parameters are dynamically replaced during the Ant build process across DEV, QA, and PRD environments.

SP-Config (ISC): Utilizing the SP-Config REST APIs to export and import IdentityNow configurations (JSON). Automating the migration of transforms, identity profiles, and provisioning policies between tenants via CI/CD pipelines.

Direct XML Manipulation: Accessing the IIQ debug page (/debug) to manipulate underlying XML registries (e.g., UIConfig, SystemConfiguration) to unhide native features, modify global timeout settings, or implement custom QuickLinks and dashboard widgets.
