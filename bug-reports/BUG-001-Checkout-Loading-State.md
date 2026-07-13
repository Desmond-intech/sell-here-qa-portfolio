# BUG-001 – Checkout Displays False Empty Cart During Initial Load

## Bug Information

| Field      | Value                                              |
| ---------- | -------------------------------------------------- |
| Bug ID     | BUG-001                                            |
| Module     | Checkout                                           |
| Feature    | Checkout Loading State                             |
| Components | `Checkout.tsx`, `Cart.tsx` (navigation validation) |
| Severity   | Medium                                             |
| Priority   | High                                               |
| Status     | Fixed                                              |

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

When users navigated from the Cart page to Checkout with products already in their cart, the Checkout page briefly displayed the message:

> "Your order is empty, fill the cart and come back."

Although the cart contained valid items, the message appeared during the initial loading phase before the checkout content was rendered.

This resulted in a misleading user experience by displaying an incorrect empty-cart state.

---

## Preconditions

- The user has one or more products in the shopping cart.
- The user accesses Checkout using the **Proceed to Checkout** button.
- The **Proceed to Checkout** button is enabled only when the cart contains products.

---

## Steps to Reproduce

1. Add one or more products to the shopping cart.
2. Open the Cart page.
3. Click **Proceed to Checkout**.
4. Observe the Checkout page immediately after navigation.

---

## Expected Result

A loading indicator should be displayed while checkout data is being initialized. Once loading is complete, the Checkout page should render without displaying an incorrect empty-cart message.

---

## Actual Result

The Checkout page briefly displayed:

> "Your order is empty, fill the cart and come back."

The correct checkout content appeared only after the cart data finished loading.

---

## Root Cause (Confirmed During Code Review)

The `Checkout.tsx` component evaluated the cart state before the cart data had finished initializing. As a result, the component temporarily interpreted the cart as empty and rendered the empty-state message before the actual cart data became available.

---

## Resolution

A dedicated loading state was introduced using a `<Spinner />`.

The Checkout component now waits for cart data to finish loading before evaluating whether the cart is empty, preventing the false empty-cart message from appearing during page initialization.

---

## Business Impact

Although the issue did not affect cart data or order processing, it displayed misleading information during a critical stage of the purchasing journey. Resolving the issue improved user confidence and provided a smoother checkout experience.

---

## Evidence

- Video 1 – Bug reproduction
- Video 2 – Fix verification

---

## Verification Status

**Result:** ✅ Fixed

The issue was successfully resolved. Manual verification confirmed that the loading spinner is displayed during data initialization and that the incorrect empty-cart message no longer appears when the cart contains products.
