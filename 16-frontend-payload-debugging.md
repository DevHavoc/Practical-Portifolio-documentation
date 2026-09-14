# Frontend Payload Debugging

## Context

During the user access creation flow, the backend returned a `NullReferenceException` inside the `CreateUserAccessCase`.

The exception occurred when accessing the request DTO:

    dto.UserId

At first, the problem appeared to be a backend issue. The request was therefore traced from the frontend to the API.

## Request Flow

    SAPUI5
      ↓
    RequestModel
      ↓
    HTTP POST
      ↓
    Controller
      ↓
    CreateUserAccessCase
      ↓
    Repository
      ↓
    PostgreSQL

The first step was to inspect the actual payload being sent by the frontend.

## Finding the Problem

A temporary `console.log` showed:

    {
      userId: null,
      system: "Office365",
      isActive: true,
      source: "Manual"
    }

The problem was therefore not the access system or repository.

The frontend was attempting to create an access without a valid `UserId`.

The value came from:

    var userId = that._dlgUserId;

The user had not yet been persisted, so the identifier was still `null`.

## Restoring the Validation

The intended lifecycle was:

    Create user
        ↓
    Save user
        ↓
    Obtain user ID
        ↓
    Add access

A validation was restored:

    if (!userId) {
      MessageToast.show("Salve o usuário antes de adicionar acessos.");
      return;
    }

This preserved the database relationship instead of allowing an invalid access record to reach the backend.

## Lessons Learned

- The location of an exception is not necessarily the location of its root cause.
- Inspecting the actual HTTP payload can quickly eliminate incorrect assumptions.
- Frontend state should be validated before changing backend behavior.
- Business-rule validations often protect important entity relationships.

## Engineering Takeaway

When debugging full-stack problems, trace the data backward until the first divergence appears.

    Backend exception
        ↑
    API payload
        ↑
    Frontend state

Finding where the data first became invalid is often more useful than fixing where the application finally crashed.