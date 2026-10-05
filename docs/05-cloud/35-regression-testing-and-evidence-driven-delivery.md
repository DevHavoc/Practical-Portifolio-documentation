# Regression Testing and Evidence-Driven Delivery

## Problem

Several changes appeared correct at one layer while remaining broken at another. Examples included a selected export returning one record, a summary DTO constructor mismatch preventing compilation, an overview appearing without permission, and a report issuing successful remote requests without completing.

A credible delivery process needed to distinguish implemented behavior, successful tests, measured performance, and unresolved limitations.

## Context

The work combined SAPUI5 state and layout, ASP.NET Core endpoints, application use cases, Entity Framework Core persistence, binary exports, and external HTTP integration. One passing test category could not establish correctness across every layer.

## Investigation

The investigations used screenshots, request payloads, HTTP status codes, compilation errors, source inspection, targeted unit tests, and read-only remote measurement.

Each type of evidence answered a different question. A compiler success did not prove a PDF contained all selected users. An HTTP 200 response did not prove the report's entire collection phase had completed. A unit test with an immediate mock response did not establish external-service latency.

## Solution

Validation was divided into complementary scopes:

| Scope | Purpose |
|---|---|
| Compilation | Identify contract, constructor, and dependency errors |
| Focused unit/regression tests | Protect calculations, scope, traversal, pagination, and error behavior |
| UI review | Inspect selection state, card geometry, toolbar placement, and icons |
| Request inspection | Verify identifiers, filters, authorization outcomes, and binary responses |
| Read-only integration measurement | Establish collection timing with the real external dependency |
| Full suite | Identify failures outside the focused change as well as regressions |

The summary-contract issue was treated as a cross-layer synchronization problem: DTO signatures and all constructing callers must agree. The initial module-loading failure was not represented as a separate successfully delivered feature.

Temporary measurement code was removed after use. Maintained regression tests were kept; removing diagnostic fixtures is not the same as deleting the tests that protect the implementation.

## Validation Record

The final recorded implementation-session results were:

- Infrastructure compilation succeeded with zero warnings and zero errors in that build.
- All 47 focused DRE/OpenProject tests passed.
- The full suite reported 706 passing tests and 15 failures, for 721 tests total.
- Those failures were in e-mail/password workflows, import tests, and null-data export expectations, outside the focused DRE tests.
- The final read-only collection measurement completed in 179.1 seconds.
- The user subsequently confirmed that the optimization worked.

No claim is made that the full suite was green. Without a separately reproduced baseline, failures outside the changed area should not automatically be called pre-existing.

These results are historical evidence from the session. Documentation generation did not rerun the whole suite or contact production to generate new reports.

## Why This Solution

Separating evidence types prevents overclaiming. It also makes remaining work explicit: successful focused tests can coexist with unrelated suite failures, and a successful local collection measurement can coexist with production variability.

## Result

The delivered documentation records the final implementation, unsuccessful experiments, and validation boundaries. It avoids claiming that every requested behavior was freshly exercised on the currently checked-out branch.

## Lessons Learned

Describe what was observed, not what the result was hoped to prove. Especially for financial reports and authorization paths, partial success or missing evidence must remain visible.

## Possible Improvements

Add automated PDF-content tests with synthetic data, negative capability-authorization tests, visual regression coverage for the shared cards, and reproducible performance runs with several dataset sizes.

Production monitoring should track stage duration, request failure rate, and cache freshness without collecting credentials or personal record content in logs.
