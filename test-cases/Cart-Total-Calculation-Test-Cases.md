# Cart Total Calculation Test Cases

## Feature

Cart Total Calculation

**Components Covered**

- `Cart.tsx`
- `CartDrawer.tsx`
- `ProceedToPayment.tsx`

---

### TC-021 – Verify Empty Cart Total

**Preconditions**

- Shopping cart is empty.

**Steps**

1. Open the Cart page.
2. Open the Cart Drawer.
3. Navigation to the checkout summary is disabled when cart is empty.

**Expected Result**

All components(excluding Checckout.tsx) display a total of **0** without rendering **NaN**.

---

### TC-022 – Verify Single Item Total Calculation

**Preconditions**

- Shopping cart contains one valid product.

**Steps**

1. Add a single product to the cart.
2. Open the Cart page.
3. Open the Cart Drawer.
4. View the Proceed to Payment summary.

**Expected Result**

All components display the same correct subtotal.

---

### TC-023 – Verify Multiple Item Total Calculation

**Preconditions**

- Shopping cart contains multiple valid products.

**Steps**

1. Add multiple products to the cart.
2. Compare the totals displayed in:
   - Cart page
   - Cart Drawer
   - Proceed to Payment summary

**Expected Result**

All components display identical accumulated totals.

---

### TC-024 – Verify Total Updates After Quantity Changes

**Preconditions**

- Shopping cart contains one or more products.

**Steps**

1. Increase the quantity of a product.
2. Verify the displayed totals.
3. Decrease the quantity.
4. Verify the displayed totals again.

**Expected Result**

Totals recalculate immediately and remain synchronized across all components.

---

### TC-025 – Verify Total Updates After Item Removal

**Preconditions**

- Shopping cart contains multiple products.

**Steps**

1. Remove a product from the cart.
2. Observe the totals in all components.

**Expected Result**

Totals recalculate correctly and remain synchronized.

---

### TC-026 – Verify Invalid Cart Entries Are Ignored

**Preconditions**

- Shopping cart contains malformed or invalid cart data.

**Steps**

1. Load the application with invalid cart entries.
2. Observe the displayed totals.

**Expected Result**

Invalid entries are safely ignored, valid entries continue to be calculated correctly, and **NaN** is never displayed.

---

### TC-027 – Verify Shared Calculation Consistency

**Preconditions**

- Shopping cart contains valid products.

**Steps**

1. Perform several cart operations, including:
   - Adding products
   - Removing products
   - Updating quantities

2. Compare totals across:
   - Cart page
   - Cart Drawer
   - Proceed to Payment summary

**Expected Result**

All components display identical totals throughout every cart operation.

---

### TC-028 – Verify Defensive Calculation Stability

**Preconditions**

- Application is running.

**Steps**

1. Execute normal cart operations.
2. Verify application behaviour throughout the session.

**Expected Result**

- No **NaN** values are displayed.
- No calculation-related rendering errors occur.
- No calculation-related console exceptions are generated.
- Checkout totals remain synchronized with the cart state.
