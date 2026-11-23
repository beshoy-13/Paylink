
## Non-functional requirements

| ID | Quality | Metric | Target | How it is measured |
|---|---|---|---|---|
| NFR-1 | Correctness | Double charges and negative balances | 0 and 0 | Duplicate-request and concurrent-payment tests (hard cases 1, 4 and 6) |
| NFR-2 | Performance | Sustained payment throughput against the mock billers | 100 payments per second for 60 seconds | Load test with k6 or NBomber in Phase 2 |
| NFR-3 | Resilience | Payments to other billers during a biller outage | 100% still succeed | Stop the ElectroCo mock, then pay an AquaCo bill |
| NFR-4 | Modifiability | Files changed outside Biller Integration and configuration when adding a biller | 0 | Count changed files in the git diff (hard case 8) |
| NFR-5 | Auditability | Money movements with an audit record (who, when, why) | 100% | Test compares the number of ledger entries with audit entries |
| NFR-6 | Testability | Line coverage on domain logic | At least 70% | Coverlet report produced by the CI pipeline |
| NFR-7 | Reproducibility | A fresh clone starts the application and the database with one command | docker compose up --build succeeds and /health returns 200 | Clean-clone test on a second machine, plus the green CI run on every push |
