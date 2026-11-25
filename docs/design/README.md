# PayLink Design Overview

## Purpose

This document records the confirmed architecture and technology decisions for the PayLink Phase 1 design. PayLink is a bill-payment and wallet backend intended to organize core business capabilities into clear, independently maintained modules within a single deployable application.

## Technology Stack

| Technology                | Purpose                                                                           |
| ------------------------- | --------------------------------------------------------------------------------- |
| C#                        | Primary implementation language for the application and its modules.              |
| .NET 10                   | Runtime and application framework for the backend implementation.                 |
| ASP.NET Core Web API      | Provides the HTTP API used by clients to access PayLink capabilities.             |
| Entity Framework Core     | Provides relational data access and persistence mapping for application modules.  |
| PostgreSQL                | Stores application data in a relational database.                                 |
| xUnit                     | Provides the automated testing framework for unit and integration testing.        |
| OpenAPI/Swagger           | Documents and describes the HTTP API exposed by the application.                  |
| Docker and Docker Compose | Package the application and support consistent local execution and service setup. |
| GitHub Actions            | Automates repository workflows such as build and test execution.                  |

## Architectural Approach

PayLink will use a **modular monolith** organized according to **Clean Architecture principles**. The application is deployed as one backend while the source code is divided into modules with clear responsibilities and boundaries.

This approach is appropriate for the current project stage because it provides explicit separation and maintainability without the operational complexity of a distributed microservices system. Modules may interact through well-defined interfaces and internal events where appropriate. Business logic remains independent from framework and infrastructure concerns.

The design also allows modules with more independent operational needs, particularly Notifications and Reconciliation, to be extracted into separate services later if necessary. Extraction should be based on future technical need rather than introduced prematurely.

## Project Modules

The system is divided into the following eight modules:

1. **Accounts & Wallet** - Manages accounts, wallets, ledger entries, holds, and top-ups.
2. **Biller Catalog** - Maintains billers and the bill-payment services available to customers.
3. **Payment Processing** - Coordinates payments, lifecycle records, idempotency keys, and audit events.
4. **Biller Integration** - Provides biller integration and manages technical integration records.
5. **Fees & Commissions** - Applies fee rules and manages commission records.
6. **Notifications** - Creates notifications and tracks delivery status.
7. **Fraud Checks** - Applies fraud rules and maintains assessment records and configurable limits.
8. **Reconciliation** - Manages settlement files, matches payment outcomes, and maintains reconciliation records and results.

## Initial Module Data Ownership

The following ownership boundaries define the intended initial design. They describe architectural responsibility and do not mean that every listed data concept has already been implemented.

| Module                 | Initial Data Ownership                                                                        | Intermediate Details                                                                                                                                                                                                          |
| :--------------------- | :-------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Accounts & Wallet**  | Account, Wallet, LedgerEntry, Hold, and TopUp.                                                | Handles user/agent profiles, verification tiers, and account blocks. Computes posted, held, and available wallet balances dynamically from ledger entries; no balance columns are stored. |
| **Biller Catalog**     | Biller, BillerService, and RequiredField.                                                     | Maintains available billers and services, including required input fields, allowed payment ranges, and active/inactive flags, with admin management support.                                                                  |
| **Payment Processing** | Payment, PaymentStatusHistory, RefundRequest, IdempotencyKey, and AuditEvent.                 | Coordinates the payment workflow, enforces allowed state transitions, handles account-scoped idempotency, and supports payment lookup, refund, and financial/administrative audit flows.                                      |
| **Biller Integration** | BillerIntegration and BillerCallback.                                                        | Isolates protocol differences across billers (REST, SOAP, async callbacks, batch files) behind adapters, manages timeouts and retry schedules, and absorbs biller API changes without affecting core logic.                   |
| **Fees & Commissions** | FeeRule, Promotion, and Commission.                                                          | Calculates service fees across different pricing models and active promotions, and tracks agent commissions earned on completed payments.                                                                                     |
| **Fraud Checks**       | FraudRule, FraudAssessment, and ConfigSetting.                                               | Runs pre-payment risk checks—such as spending limits, rapid-fire transaction rates, and blocked accounts—and logs the evaluation results. Admin-editable limits are stored in ConfigSetting.                                  |
| **Notifications**      | Notification.                                                                                | Sends customer alerts (SMS, push, email) via templates on payment outcomes, retries failed deliveries, and tracks delivery status independently of payment success.                                                           |
| **Reconciliation**     | SettlementFile, ReconciliationRun, and ReconciliationItem.                                   | Matches internal payment records against biller settlement files, helps resolve uncertain states where rules permit, and logs discrepancies for support review.                                                             |

Audit events are a cross-cutting concern owned by the existing Payment Processing module; they do not form a ninth module.

## Core Architectural Rules

- **Dependency direction:** Dependencies point inward toward the core business logic. Infrastructure, frameworks, and external systems are outside the core and must not drive or contaminate business rules.
- **Separation of concerns:** Each module keeps its API, application behavior, domain logic, and infrastructure concerns appropriately separated. Database and framework details remain replaceable implementation concerns.
- **Module boundaries:** Modules expose only the interfaces required by other modules and do not reach directly into another module's implementation. Contracts are owned inward by the domain modules; domain logic does not depend on application or infrastructure layers.
- **Data ownership:** Each module owns and manages its own data. No module should read or modify another module's tables or persistence context directly.
- **Internal collaboration:** Modules communicate through well-defined interfaces and internal events where asynchronous or decoupled communication is appropriate.
- **Future extraction:** Module contracts and boundaries should be kept sufficiently clear that Notifications, Reconciliation, or another module may be extracted later if needed, while no such service split is implemented as a default Phase 1 decision.

## Design Documents

The following documents and diagrams are maintained in this directory:

- `README.md` - Describes the confirmed technology stack, architectural approach, modules, and core rules.
- `use-case-diagram.puml` - Records user, scheduler, and external-system interactions with PayLink using PlantUML's native use-case notation.
- `domain-class-diagram.puml` - Shows the major domain classes and relationships within and across modules.
- `er-diagram.puml` - Shows persistent entities, derived wallet balances, and their relationships.
- `architecture-diagram.puml` - Shows the modular monolith structure, module boundaries, dependencies, table ownership, and external infrastructure.
