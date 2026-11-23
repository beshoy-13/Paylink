# PayLink Design Overview

## Purpose

This document records the confirmed architecture and technology decisions for the PayLink Phase 1 design. PayLink is a bill-payment and wallet backend intended to organize core business capabilities into clear, independently maintained modules within a single deployable application.

## Technology Stack

| Technology | Purpose |
| --- | --- |
| C# | Primary implementation language for the application and its modules. |
| .NET 10 | Runtime and application framework for the backend implementation. |
| ASP.NET Core Web API | Provides the HTTP API used by clients to access PayLink capabilities. |
| Entity Framework Core | Provides relational data access and persistence mapping for application modules. |
| PostgreSQL | Stores application data in a relational database. |
| xUnit | Provides the automated testing framework for unit and integration testing. |
| OpenAPI/Swagger | Documents and describes the HTTP API exposed by the application. |
| Docker and Docker Compose | Package the application and support consistent local execution and service setup. |
| GitHub Actions | Automates repository workflows such as build and test execution. |

## Architectural Approach

PayLink will use a **modular monolith** organized according to **Clean Architecture principles**. The application is deployed as one backend while the source code is divided into modules with clear responsibilities and boundaries.

This approach is appropriate for the current project stage because it provides explicit separation and maintainability without the operational complexity of a distributed microservices system. Modules may interact through well-defined interfaces and internal events where appropriate. Business logic remains independent from framework and infrastructure concerns.

The design also allows modules with more independent operational needs, particularly Notifications and Reconciliation, to be extracted into separate services later if necessary. Extraction should be based on future technical need rather than introduced prematurely.

## Project Modules

The system is divided into the following eight modules:

1. **Accounts & Wallet** - Manages accounts, wallets, ledger entries, and holds.
2. **Biller Catalog** - Maintains billers and the bill-payment services available to customers.
3. **Payment Processing** - Coordinates payments and their lifecycle records.
4. **Biller Integration** - Provides biller integration and manages technical integration records.
5. **Fees & Commissions** - Applies fee rules and manages commission records.
6. **Notifications** - Creates notifications and tracks delivery status.
7. **Fraud Checks** - Applies fraud rules and maintains assessment records.
8. **Reconciliation** - Matches payment outcomes and maintains reconciliation records and results.

## Initial Module Data Ownership

The following ownership boundaries define the intended initial design. They describe architectural responsibility and do not mean that every listed data concept has already been implemented.

| Module | Initial data ownership |
| --- | --- |
| Accounts & Wallet | Accounts, wallets, ledger entries, and holds. |
| Biller Catalog | Billers and available bill-payment services. |
| Payment Processing | Payments and payment lifecycle records. |
| Biller Integration | Biller integration configurations and technical integration records. |
| Fees & Commissions | Fee rules and commission records. |
| Fraud Checks | Fraud rules and assessment records. |
| Notifications | Notification records and delivery status. |
| Reconciliation | Reconciliation records and results. |

## Core Architectural Rules

- **Dependency direction:** Dependencies point inward toward the core business logic. Infrastructure, frameworks, and external systems are outside the core and must not drive or contaminate business rules.
- **Separation of concerns:** Each module keeps its API, application behavior, domain logic, and infrastructure concerns appropriately separated. Database and framework details remain replaceable implementation concerns.
- **Module boundaries:** Modules expose only the interfaces required by other modules and do not reach directly into another module's implementation. Direct in-process calls and internal events are to use explicit contracts.
- **Data ownership:** Each module owns and manages its own data. No module should read or modify another module's tables or persistence context directly.
- **Internal collaboration:** Modules communicate through well-defined interfaces and internal events where asynchronous or decoupled communication is appropriate.
- **Future extraction:** Module contracts and boundaries should be kept sufficiently clear that Notifications, Reconciliation, or another module may be extracted later if needed, while no such service split is implemented as a default Phase 1 decision.

## Design Documents

The following documents and diagrams are maintained in this directory:

- `README.md` - Describes the confirmed technology stack, architectural approach, modules, and core rules.
- `use-case-diagram.mmd` - Records user and external-system interactions with PayLink.
- `domain-class-diagram.mmd` - Shows the major domain concepts and relationships within and across modules.
- `er-diagram.mmd` - Represents persistent entities and their relationships.
- `architecture-diagram.mmd` - Illustrates the modular monolith structure, module boundaries, dependencies, and external infrastructure.