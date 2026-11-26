# SupplyChain engineering guidelines

These requirements apply to the backend, frontend, and worker. The Java 8 baseline in [architecture.md](architecture.md) describes the reviewed source snapshot, not the standard for future changes.

## Java and Spring Boot baseline

- Use **Java 21 or later**. Set the Maven compiler's `release` to the project's supported Java version (at least 21) and use that version in local builds, CI, and runtime images. Verify framework, dependency, and servlet container compatibility when upgrading.
- Use **Spring Boot 4.2.x** on **JDK 21 or 25**. As of November 26, 2026, [Spring Boot 4.2.0](https://github.com/spring-projects/spring-boot/releases/tag/v4.2.0) is the latest stable release.

## Java coding standards

| Required practice | Prohibited practice |
| --- | --- |
| Use `UpperCamelCase` for types, `lowerCamelCase` for methods and variables, and `UPPER_SNAKE_CASE` for constants. Use the project's formatter and explicit imports. | Wildcard imports, unused imports, and inconsistent formatting. |
| Keep methods focused and names descriptive. Use constants for meaningful thresholds and configuration for deployment-specific values. | Duplicated business rules, unexplained magic values, and commented-out code. |
| Use typed collections and prefer immutable data. Use `BigDecimal` with explicit rounding rules for monetary calculations. | Raw generic types, unchecked casts used to bypass type safety, and `float` or `double` for money. |
| Use constructor injection for required dependencies. Keep controllers and queue listeners thin; put business rules in services. | Field injection, hidden mutable global state, and business logic duplicated across controllers or listeners. |
| Validate input at system boundaries. Catch the exceptions you can handle, preserve the cause when wrapping, and use parameterized logging without secrets or personal data. | Empty catch blocks, broad catches that hide errors, and returning success or dummy data after an unexpected failure. |
| Keep credentials in approved secret storage. Use parameterized queries and close owned resources with try-with-resources. | Hardcoded credentials, logging tokens, building SQL by concatenating user input, and leaking resources. |

## Unit testing and coverage

- Each Java project must maintain **at least 75% line coverage from unit tests** on production code. Measure it with JaCoCo or an equivalent tool, and make CI fail below 75%. Report integration and UI test coverage separately; it does not count toward this minimum.
- Use JUnit 5 for focused, deterministic tests. Cover business rules, boundary values, and expected failure paths. Mock external dependencies where needed; do not rely on live services, timing sleeps, or shared test state.
- Add a regression test for each bug fix. Do not inflate coverage with assertion-free tests or exclude business logic to pass the threshold.

## Automated UI testing

- Every web application must have automated browser tests (for example, Playwright or Selenium) that run in pull request CI. Failing tests block merge.
- Cover critical user journeys and their important failure paths. For SupplyChain, cover dashboard loading, valid order submission, validation feedback, and the order appearing in the pending list. Cover inventory browsing and approval when the corresponding UI flows are available.
- Use isolated test data, stable selectors, and condition-based waits rather than fixed sleeps. Assert outcomes the user can see, and retain traces or screenshots on failure.

## Additional Updates

- **UPDATE Nov 26:** All apps should use Azure Application Insights for monitoring.
