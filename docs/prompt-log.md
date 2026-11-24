# Prompt log

## Logging Guidelines for Team Members

- **When to Log:** Every team member must record AI interactions when an AI tool generates more than half of a change, or when an AI suggestion is analyzed, rejected, or modified.

| Date       | Author     | Stage  | Tool and model    | Prompt                                                                            | Summary of answer                                       | Verdict and why                                                                                                       |
| ---------- | ---------- | ------ | ----------------- | --------------------------------------------------------------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| 2026-11-23 | Beshoy (C) | DevOps | Claude Sonnet 5.5 | Fix the NU1903 vulnerability warning for Microsoft.OpenApi in the Web API project | Add the Microsoft.OpenApi package with no version       | Rejected: it installed 3.11.1, the template's generator needs 2.x, and the build failed with CS0200                   |
| 2026-11-23 | Beshoy (C) | DevOps | Claude Sonnet 5.5 | Update the package that pulls in Microsoft.OpenApi                                | Add Microsoft.AspNetCore.OpenApi with no version        | Rejected: it installed the .NET 11 build, the compiler could not use the analyzer (CS9057) and AddOpenApi disappeared |
| 2026-11-23 | Beshoy (C) | DevOps | Claude Sonnet 5.5 | Same fix, pinned to the .NET 10 line                                              | Add Microsoft.AspNetCore.OpenApi with --version 10.0.\* | Accepted: 10.0.15 resolved Microsoft.OpenApi 2.13.0, the warning disappeared and the build passes                     |
| 2026-11-23 | Beshoy (C) | DevOps | Claude Sonnet 5.5 | Write the GitHub Actions workflow for build, format check, test and Docker build  | Used actions/checkout v4 and actions/setup-dotnet v4    | Fixed: the release pages show checkout v7.0.1 and setup-dotnet v6.1.0, and the AI could not verify versions by search |
| 2026-11-23 | Beshoy (C) | DevOps | Claude Sonnet 5.5 | Branch protection on a private repo with GitHub Free                              | Suggested making the repo public as a fallback          | Fixed: the brief requires a private repo, so the TA must be told and the repo switched back after the Student Pack    |
