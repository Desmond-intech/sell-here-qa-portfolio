# Cart Interaction Test Cases

## Feature

Proceed to Payment Panel

---

### TC-014 – Verify Interaction with Cart Controls During Payment Panel Overlap

**Preconditions**

- User has one or more products in the shopping cart.
- Desktop viewport is active.
- Cart page is open.

**Steps**

1. Open the Cart page.
2. Scroll until one or more cart items are positioned parallel to the fixed **Proceed to Payment** panel.
3. Attempt to interact with:
   - Remove
   - Favorite
   - Quantity controls (+ / −)

**Expected Result**

All controls remain fully interactive and execute their intended actions while positioned parallel to the fixed payment panel.

---

### TC-015 – Verify Interaction Outside the Payment Panel Area

**Preconditions**

- User has one or more products in the shopping cart.
- Desktop viewport is active.

**Steps**

1. Open the Cart page.
2. Position the cart items so they are **not** parallel to the fixed payment panel.
3. Attempt to interact with:
   - Remove
   - Favorite
   - Quantity controls (+ / −)

**Expected Result**

All controls remain fully interactive and execute their intended actions.

---

### TC-016 – Verify Empty Cart Interaction

**Preconditions**

- Shopping cart is empty.
- Desktop viewport is active.

**Steps**

1. Open the Cart page with an empty shopping cart.
2. Observe the page immediately after it loads.
3. Attempt to click the **Start Shopping** button.

**Expected Result**

The **Start Shopping** button remains fully interactive even though it is positioned parallel to the fixed **Proceed to Payment** panel.

---

### TC-017 – Verify Pointer Feedback During Payment Panel Overlap

**Preconditions**

- Desktop viewport is active.
- Cart page is open.

**Steps**

1. Position an interactive element parallel to the fixed payment panel.
2. Hover the mouse pointer over the interactive control.

**Expected Result**

The pointer changes appropriately, indicating that the control is interactive.

---

### TC-018 – Verify Scrolling Does Not Affect Cart Interaction

**Preconditions**

- User has products in the shopping cart.

**Steps**

1. Scroll through the Cart page several times.
2. Position different cart items parallel to the payment panel.
3. Repeat interaction with the available controls.

**Expected Result**

Scrolling does not affect the ability to interact with cart controls regardless of their position relative to the payment panel.
