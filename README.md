<div align="center">

# ⚡ PayLink

**A modular monolith bill payment & wallet settlement platform built with .NET 10**

![.NET 10](https://img.shields.io/badge/.NET-10.0%20LTS-512BD4?style=flat-square&logo=dotnet)
![Architecture](https://img.shields.io/badge/Architecture-Hexagonal%20Modular%20Monolith-cyan?style=flat-square)
![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL%2017-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Container-Docker%20Compose-2496ED?style=flat-square&logo=docker&logoColor=white)
![CI](https://img.shields.io/badge/CI-GitHub%20Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white)

</div>

---

## 🖥 System Overview

| Component | Choice |
|---|---|
| Runtime / Framework | C# on .NET 10 (LTS) — ASP.NET Core Web API |
| Architecture | Hexagonal (Ports & Adapters) Modular Monolith |
| Database / ORM | PostgreSQL 17 + Entity Framework Core (Code-First Migrations) |
| Money Representation | Exact `decimal` (`decimal(18,2)`), ledger-computed balances |
| Async & Messaging | In-process domain events (Phase 1) → RabbitMQ / MassTransit (Phase 2) |
| API Documentation | Built-in OpenAPI / Swashbuckle (20+ endpoints) |
| Testing & Coverage | xUnit, FluentAssertions, NSubstitute/Moq, Testcontainers, Coverlet (≥ 70%) |
| Linting & Formatting | `dotnet format` + Roslyn Analyzers |
| Infrastructure & CI | Multi-stage Dockerfile, Docker Compose, GitHub Actions |

---

## 📁 Repository Structure

paylink/
├── .github/
│   ├── workflows/
│   │   └── ci.yml                      # Restore, format check, build, test coverage & Docker build
│   └── pull_request_template.md        # PR checklist, AI disclosure & peer-review verification
├── docs/
│   ├── requirements.md                 # Problem summary, stakeholders, FR-1–28 user stories & NFRs
│   ├── prompt-log.md                   # AI usage log with prompt iterations & rejected outputs
│   ├── design/                         # PlantUML / Mermaid source diagrams (Use Case, Class, ER, Seq, State)
│   └── adr/                            # Architecture Decision Records (Phase 2)
├── src/
│   ├── PayLink.Api/                    # ASP.NET Core host (DI wiring & endpoints only, no business logic)
│   ├── PayLink.SharedKernel/           # Money value object, base primitives & integration event contracts
│   ├── PayLink.Accounts/               # Users, wallets, double-entry ledger entries & balance holds
│   ├── PayLink.Catalog/                # Billers, biller services & dynamic input field definitions
│   ├── PayLink.Payments/               # Payment lifecycle state machine, idempotency & orchestration
│   ├── PayLink.BillerIntegration/      # Outbound adapters & factories for external biller APIs
│   ├── PayLink.Fees/                   # Fee calculation strategies & agent commission rules
│   ├── PayLink.Notifications/          # SMS/Push notifications & retry command handlers
│   ├── PayLink.Fraud/                  # Chain-of-Responsibility fraud & velocity rule checks
│   └── PayLink.Reconciliation/         # Settlement file generation & discrepancy reconciliation
├── tests/
│   ├── PayLink..Tests/         # Isolated unit tests per domain module
│   └── PayLink.Api.IntegrationTests/   # End-to-end & hard-case concurrency tests (Testcontainers)
├── mocks/                              # Mock biller servers: ElectroCo, AquaCo, TopUpNet (Phase 2)
├── Dockerfile                          # Multi-stage .NET 10 SDK → ASP.NET runtime build
├── docker-compose.yml                  # API + PostgreSQL 17 with database healthcheck
├── Directory.Build.props               # Shared compiler settings, nullable & analyzer enforcement
├── global.json                         # Pinned .NET 10 SDK version
├── .editorconfig                       # Consistent code style rules across IDEs and CI
├── .gitattributes                      # Enforces LF line endings for Linux container compatibility
├── .env.example                        # Template for local environment secrets
└── PayLink.sln                         # Solution file


---

## ⚙️ Architecture & Core Highlights

- **Hexagonal Modular Monolith** — 9 domain modules communicating through public interfaces (ports) or events rather than each other's internal classes.
- **Ledger-First Financial Integrity** — Wallet available and held balances are computed from ledger entries and holds (never stored as a balance column), using exact `decimal` (`decimal(18,2)`) for money.
- **Strict Payment State Machine** — Covers the 9 payment transitions (`Initiated` to `FundsHeld` or `Rejected`, `FundsHeld` to `SentToBiller`, `SentToBiller` to `Completed`, `Failed`, or `Unknown`, `Unknown` to `Completed` or `Failed`, and `Failed` to `Refunded`), keeping `Completed`, `Rejected`, and `Refunded` final and preventing `Unknown` from going directly to `Refunded`.
- **Idempotency & Concurrency Control** — Uses a unique constraint/index on the idempotency key, database transactions, and optimistic concurrency on wallets to prevent double charges or negative balances.
- **Design Patterns (Phase 2)** — State (payment lifecycle), Chain of Responsibility (fraud rules), Adapter & Factory (biller integrations), Strategy (fees), Observer (notifications), and Command (retries).
- **Resilient Biller Isolation** — Timeouts or outages in one biller (such as ElectroCo) transition payments to `Unknown` while keeping other billers (like AquaCo and TopUpNet) working.

---

## 📊 Non-Functional Targets

| Quality Attribute | Target | Verification Method |
|---|---|---|
| **Correctness** | 0 double charges & 0 negative balances | Duplicate-request & concurrent-payment tests (hard cases 1, 4, 6) |
| **Performance** | 100 payments/sec sustained for 1 minute | Load test (k6 / NBomber) against mock billers in Phase 2[cite: 1] |
| **Resilience** | A biller outage does not stop payments to other billers[cite: 1] | Stop ElectroCo mock, then pay an AquaCo bill[cite: 1] |
| **Modifiability** | Adding a biller changes only Biller Integration and configuration[cite: 1] | Count files changed outside that module in git diff (target: 0)[cite: 1] |
| **Auditability** | 100% of money movements have an audit record with who, when and why[cite: 1] | Test compares ledger entries with audit entries[cite: 1] |
| **Testability** | At least 70% line coverage on domain logic[cite: 1] | Coverlet report produced by the CI pipeline[cite: 1] |

---

## 🚀 Getting Started

```bash
git clone [https://github.com/](https://github.com/)<owner>/paylink.git
cd paylink
cp .env.example .env
docker compose up --build
