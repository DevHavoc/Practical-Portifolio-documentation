# Production Debugging Methodology

## Principle

Effective debugging is not random code inspection.

It is a process of reducing uncertainty until the first incorrect assumption or state transition is identified.

## The Method

```text
1. Reproduce
      ↓
2. Collect evidence
      ↓
3. Identify the first failing layer
      ↓
4. Inspect inputs and state
      ↓
5. Form a hypothesis
      ↓
6. Apply the smallest justified fix
      ↓
7. Validate
      ↓
8. Check for regression
```

## 1. Reproduce

First determine whether the behavior can be reproduced consistently.

Record:

- exact operation;
- input;
- endpoint;
- user context;
- environment;
- expected behavior;
- actual behavior.

## 2. Collect Evidence

Useful sources include:

- browser Network tab;
- application logs;
- breakpoints;
- exception messages;
- HTTP status codes;
- database state;
- generated SQL;
- Git history;
- deployment logs.

Evidence is more reliable than intuition.

## 3. Find the First Failing Layer

For a full-stack feature:

```text
UI
 ↓
Request
 ↓
Controller
 ↓
Use Case
 ↓
Repository
 ↓
EF Core
 ↓
Database
```

Ask:

> Where does the expected behavior stop being true?

That point is usually more valuable than the final error message.

## 4. Inspect State

Once the failing layer is known, inspect its state.

Examples:

```text
Wrong request body?
Wrong route?
Null dependency?
Detached entity?
Missing AddRange?
Wrong database?
Stale branch?
Missing package source?
```

## 5. Hypothesis

A good hypothesis should be testable.

Bad:

> "EF is being weird."

Better:

> "The entities are created but never attached to the DbContext, so SaveChanges has no Added entries."

## 6. Smallest Justified Fix

Avoid unrelated refactoring during incident debugging.

The goal is:

```text
Known cause
 ↓
Minimal correction
 ↓
Validation
```

Once the system is stable, architectural improvements can be considered separately.

## 7. Validate

Validation should include more than "the error disappeared."

Check:

- expected HTTP response;
- expected database state;
- related functionality;
- logs;
- deployment/build;
- regression behavior.

## Example: Deployment Failure

A deployment with dozens of errors can be investigated as:

```text
NU1301
 ↓
Inspect NuGet configuration
 ↓
Find local package source
 ↓
Disable/remove invalid source
 ↓
New errors appear
 ↓
Check deployment branch
 ↓
Branch is stale
 ↓
Synchronize source state
 ↓
Build succeeds
```

The important insight is that the first visible error was not necessarily the final root cause of the entire deployment failure.

## Lessons Learned

- Debugging is a reasoning process.
- The first failing layer is more useful than the loudest symptom.
- Hypotheses should be testable.
- Minimal fixes reduce collateral risk.
- Validation must verify the intended state, not merely the absence of an exception.

## Engineering Takeaway

Strong debugging is the ability to turn a vague failure into a sequence of **observable, testable states**.
