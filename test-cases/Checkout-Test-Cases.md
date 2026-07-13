# Checkout Test Cases

## Feature

Checkout Loading State

---

### TC-001 – Loading Indicator

**Preconditions**

- User has one or more products in the shopping cart.
- User navigates to Checkout using the **Proceed to Checkout** button.

**Steps**

1. Add products to the cart.
2. Click **Proceed to Checkout**.
3. Observe the page immediately after navigation.

**Expected Result**

- A loading spinner is displayed while checkout data is being initialized.

---

### TC-002 – Checkout Loads Successfully

**Preconditions**

- Checkout loading has completed.

**Steps**

1. Wait for the loading spinner to disappear.

**Expected Result**

- The Checkout page displays the products in the cart and all checkout information correctly.

---

### TC-003 – False Empty-State Removed

**Preconditions**

- User has products in the shopping cart.

**Steps**

1. Refresh the Checkout page.
2. Observe the initial page render.

**Expected Result**

- The message **"Your order is empty, fill the cart and come back."** is never displayed while the cart contains products.

---

### TC-004 – Multiple Refresh Validation

**Preconditions**

- User has products in the shopping cart.

**Steps**

1. Refresh the Checkout page several times.
2. Observe the UI during each load.

**Expected Result**

- The loading spinner displays consistently before checkout content loads.
- No incorrect empty-cart message or UI flicker is observed.

---

### TC-005 – Cart Navigation Validation

**Preconditions**

- Cart contains no products.

**Steps**

1. Open the Cart page.
2. Observe the **Proceed to Checkout** button.

**Expected Result**

- The **Proceed to Checkout** button is disabled, preventing navigation to Checkout.

---

### TC-006 – Direct Checkout Route Protection

**Preconditions**

- Cart is empty.
- User is not an administrator.

**Steps**

1. Manually enter the Checkout URL in the browser.

**Expected Result**

- The application redirects the user to the Cart page.
