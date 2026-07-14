# BUG-003 – NaN Displayed Across Cart Total Calculations

## Bug Information

| Field      | Value                                                |
| ---------- | ---------------------------------------------------- |
| Bug ID     | BUG-003                                              |
| Module     | Cart & Checkout                                      |
| Feature    | Cart Total Calculation                               |
| Components | `Cart.tsx`, `CartDrawer.tsx`, `ProceedToPayment.tsx` |
| Severity   | High                                                 |
| Priority   | High                                                 |
| Status     | Fixed                                                |

---

## Environment

| Item             | Value                     |
| ---------------- | ------------------------- |
| Application      | Sell Here                 |
| Browser          | Google Chrome             |
| Operating System | Windows 10                |
| Device           | Desktop                   |
| Test Type        | Manual Functional Testing |

---

## Description

Cart subtotal calculations displayed **NaN (Not a Number)** across multiple areas of the application, including the Cart page, Cart Drawer, and Proceed to Payment panel.

Because all three components relied on the same shared calculation logic, the defect propagated consistently wherever cart totals were displayed.

This resulted in incorrect subtotal values and an unreliable checkout experience.

---

## Preconditions

- The application is running.
- Cart totals are calculated using the shared cart total utility.
- The cart contains one or more entries.

---

## Steps to Reproduce

1. Populate the shopping cart.
2. Perform cart operations such as:
   - Increasing quantity
   - Decreasing quantity
   - Removing items

3. Observe the displayed totals in:
   - Cart page
   - Cart Drawer
   - Proceed to Payment panel

---

## Expected Result

All cart totals are calculated correctly and remain synchronized across every component that consumes the shared calculation logic.

---

## Actual Result

One or more cart total displays rendered **NaN**, indicating that invalid values had entered the calculation pipeline.

---

## Root Cause (Confirmed During Code Review)

The shared cart total calculation accepted invalid cart entries into the calculation pipeline. Insufficient validation allowed malformed data—such as invalid keys, undefined entries, invalid price values, and invalid quantity values—to reach the subtotal calculation, ultimately producing **NaN** results.

---

## Resolution

The shared calculation logic was strengthened by validating cart entries before they entered the calculation pipeline. Only valid cart objects are now processed, while malformed entries are safely ignored. Additional defensive validation was introduced during subtotal calculation to prevent invalid data from affecting displayed totals.

---

## Business Impact

This issue affected multiple user-facing components responsible for displaying cart totals. Because the defect originated in shared logic, it propagated throughout the Cart and Checkout experience, reducing user confidence in pricing accuracy.

---

## Evidence

- Screenshots showing **NaN** before the fix.
- Screenshots showing correct totals after the fix.
- Console output used during investigation (if applicable).

---

## Verification Status

**Result:** ✅ Fixed

Manual verification confirmed that all components consuming the shared cart calculation display accurate, synchronized totals under normal and defensive test scenarios.
