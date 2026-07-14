# BUG-002 – Fixed Payment Panel Blocks Cart Page Interaction

## Bug Information

| Field      | Value                              |
| ---------- | ---------------------------------- |
| Bug ID     | BUG-002                            |
| Module     | Cart                               |
| Feature    | Proceed to Payment Panel           |
| Components | `ProceedToPayment.tsx`, `Cart.tsx` |
| Severity   | High                               |
| Priority   | High                               |
| Status     | Fixed                              |

---

## Environment

| Item              | Value                     |
| ----------------- | ------------------------- |
| Application       | Sell Here                 |
| Browser           | Google Chrome             |
| Operating System  | Windows 10                |
| Screen Resolution | 1920 × 1080               |
| Device            | Desktop                   |
| Test Type         | Manual Functional Testing |

---

## Description

On large screen resolutions, the **Proceed to Payment** panel remained fixed while the user scrolled through the Cart page. When the panel overlapped interactive elements, it intercepted pointer events and prevented users from interacting with the controls horizontally aligned (parallel) with it..

This affected several key user actions on the Cart page and created the impression that the application had become unresponsive.

---

## Preconditions

- User has products in the shopping cart.
- Desktop viewport is active.
- Cart page is open.

- User has no products in the shopping cart.
- Desktop viewport is active.
- Cart page is open.
- Start Shopping button is exaclty parallel to the panel.

---

## Steps to Reproduce

Scenario 1

1. Add one or more products to the cart.
2. Navigate to the Cart page.
3. Scroll until the **Proceed to Payment** panel overlaps the product list.
4. Attempt to interact with:
   - Remove button
   - Favorite button
   - Quantity controls (+ / −) buttons

Scenario 2

1. There are no products in the cart.
2. Navigate to the Cart page.
3. Scroll until the **Proceed to Payment** panel overlaps the start shopping button.
4. Attempt to interact with:
   - Start shopping Button

---

## Expected Result

Interactive controls remain accessible regardless of the position of the payment panel.

---

## Actual Result

When the payment panel overlapped the product list or start shhopping button:

- Buttons became unresponsive.
- Cursor did not change to a pointer over affected controls.
- Click handlers and navigation did not execute.

When no overlap occurred, all controls functioned normally.

---

## Root Cause (Confirmed During Investigation)

The **Proceed to Payment** component used `position: fixed`, creating a floating element above the page content. When this element overlapped the cart items or start shopping button, it intercepted pointer events and prevented interaction with the underlying controls.

---

## Resolution

The layout was updated to eliminate the interaction conflict by replacing the overlapping fixed-position behavior with a layout that no longer blocked user interaction.

---

## Business Impact

This issue affected core shopping functionality by preventing users from removing products, managing favourites, or continuing shopping whenever the payment panel overlapped the cart content. Resolving the issue restored normal interaction and improved the overall usability of the Cart page.

---

## Evidence

- Video 1 – Bug reproduction
- Video 2 – Fix verification

---

## Verification Status

**Result:** ✅ Fixed

Manual verification confirmed that all Cart page controls remain fully interactive regardless of page scroll position or payment panel visibility.
Mobile view not affected.
