# Summary Cards, Result Counts and Pagination

## Problem

Users and Access Assignments needed an at-a-glance overview, but the first card layouts suffered from clipped content, horizontal scrolling, narrow progress bars, inconsistent icon alignment, and intrusive framework focus styling.

The list also needed to distinguish total registered records from filtered results instead of displaying only the current page size.

## Context

The design was iterated against reference images and an existing dashboard module. The final direction was a single row of compact cards with rounded corners, semantic icons, distinct colors, useful proportions, and click behavior.

Requirements evolved during the iteration. A proposed Without Access card was replaced with Common Users. Total cards lost their redundant full progress bar and percentage. Pagination ultimately defaults to 25, not the briefly requested 50.

## Investigation

The layout was examined at several layers: outer list item, content wrapper, inner card, icon container, and progress component. Rounding only the inner card left a square outer shadow or background visible.

A progress component set to full width could still collapse when its flex ancestors constrained it. Similarly, fixed minimum card widths could push the sixth card outside the viewport even when the row appeared correctly aligned at its left edge.

Counts required separate definitions. Current-page rows, filtered total, overall total, and summary-card metrics answer different questions.

## Solution

Shared scoped styles support a six-card row in both screens. Flexible equal-width items, shrinkable containers, matching rounded wrappers, and a full-width footer address overflow and progress-bar sizing. Icon containers center the confirmation and decline symbols; the warning icon receives a small optical vertical adjustment.

Distinct semantic colors identify categories, with blue reserved for the registered-total card. Total cards show the absolute count without a full bar or a 100% badge. Other cards retain their percentage and directional-style icon.

Summary queries aggregate the dataset on the backend rather than counting the current page. The UI handles an empty denominator without division by zero.

## Metric and Interaction Contract

| Screen | Card | Click behavior |
|---|---|---|
| Users | Registered users | Clear status/profile/position restrictions |
| Users | Active or inactive | Apply corresponding status |
| Users | Administrators or common users | Apply corresponding profile |
| Users | New in the last 30 days | Order by creation date, newest first |
| Assignments | Registered assignments | Clear status restriction |
| Assignments | Active, inactive, or suspended | Apply corresponding status |
| Assignments | Users with assignments | Order by user |
| Assignments | Represented systems | Order by system |

These shortcuts return to the first page. They do not necessarily clear unrelated search or classification criteria. In particular, the recent-users card orders the list; it does not implement a strict last-30-days filter.

For Users, category percentages use total registered users as the denominator. For Assignments, status percentages use total assignments. The inspected implementation also displays distinct-user and distinct-system counts relative to total assignments; those ratios have different units and must not be presented as population coverage or adoption rates.

The arrow is decorative proportion styling. No historical comparison was introduced, so it must not be described as measured growth.

## Result Counts and Page Sizes

The list receives an overall total separately from the filtered total:

```text
No active filter: (35 users)
Active filter:    12 of 35 users
```

These numbers are synthetic examples. Assignment counts follow the same display rule. The filtered total is the whole matching result set, not the number of rows on the current page.

Both screens offer 10, 25, 50, and 100 records per page, with 25 initially selected. Changing the size resets pagination.

## Validation

Historical code inspection confirms the summary DTOs, repository aggregations, card actions, result-count contract, page-size options, and final default. The chat records successive visual corrections and acceptance of the shared card model.

Visual regression checks should include the rightmost card, all wrapper corners, icon centers, bar width, zero records, and multiple viewport sizes. Earlier constructor-argument errors also illustrate why summary DTOs and all their callers must evolve together.

## Result

Both administration screens have a consistent overview and more useful navigation shortcuts. List counts remain meaningful when filtering or paging, and administrators can choose a larger page without losing the initial 25-record default.

## Lessons Learned

A visual KPI needs a documented numerator, denominator, time window, and click behavior. Good-looking arrows and bars should not imply analytics that the backend does not calculate.

## Possible Improvements

Add a real recent-date filter and historical trend comparisons only with explicit backend support. Test a reduced-width layout and a visible, unobtrusive keyboard-focus indicator: removing an unwanted blue border alone does not establish accessibility.
