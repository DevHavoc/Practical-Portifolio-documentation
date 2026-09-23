# SAPUI5 Row Interaction

## Context

The Users screen displays users through a SAPUI5 `sap.m.Table`.

Initially, the user overview was opened by double-clicking only the name field.

Although functional, this made the interaction dependent on a specific cell even though the overview represented the entire user.

## Changing the Interaction Scope

The interaction was moved from the individual name control to the `ColumnListItem`.

The flow becomes:

    Table row
        ↓
    Double click
        ↓
    Get binding context
        ↓
    Get user object
        ↓
    Open overview

The row already contains the user's binding context, so no specific cell needs to be targeted.

The relevant logic is based on:

    var oContext = oItem.getBindingContext(USERS_MODEL);
    var oUser = oContext && oContext.getObject();

This keeps the interaction tied to the entity represented by the row.

## Preventing Duplicate Events

The table can be updated multiple times.

To avoid attaching multiple double-click handlers to the same row, a small flag is used:

    if (oItem._dblClickBound) {
      return;
    }

    oItem._dblClickBound = true;

This ensures that the event is registered only once for each rendered row.

## Interaction Responsibilities

The row-level overview does not replace the existing action buttons.

    Double click row → Inspect user

    Edit button      → Edit user

    Disable button   → Change status

    Delete button    → Delete user

Each interaction therefore has a different responsibility.

## Lessons Learned

- Interaction scope should match the scope of the action.
- A table row often represents the entity more naturally than an individual cell.
- Binding actions to the row context reduces dependence on the table's visual layout.
- Dynamic UI updates require care to avoid registering duplicate events.

## Engineering Takeaway

When an action applies to an entire entity, the UI interaction should generally represent that same scope.

This makes the interface more natural while keeping explicit modification actions separate.